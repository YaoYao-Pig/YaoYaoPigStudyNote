# CS的设计

服务器的设计是这样的一个思路：

## 初始化TcpListener

包括了创建Socket对象

## 等待注册

首先服务器启动注册和等待流程，然后等待用户注册的流程

也就是通过监听端口，监听创建试图连接到服务器的客户端

```c#
//NetworkProxy
public void Awake(NetworkProtocol protocol, IPEndPoint ipEndPoint){
    try {
        switch (protocol) {
            case NetworkProtocol.TCP:
                this.Service = new TService(ipEndPoint);
                break;
            default:
                throw new ArgumentOutOfRangeException();
        }

        this.StartAccept(); //<--注意这个
    }
    catch (Exception e) {
        throw new Exception($"{ipEndPoint}", e);
    }
}
//他会不断的循环，检查客户端对于服务器链接的请求
private async void StartAccept(){
    while (true) { //<--注意这个循环
        if (this.IsDisposed) {
            return;
        }

        await this.Accept();
    }
}
public virtual async Task<Session> Accept(){
    AChannel channel = await this.Service.AcceptChannel(); //<--注意这个挂起
    Session session = CreateSession(this, channel);
    channel.ErrorCallback += (c, e) => { this.Remove(session.Id); };
    this.sessions.Add(session.Id, session);
    session.Start();
    return session;
}
```

## 开始更新（服务器）

服务器会不断的接受来自客户端的消息

```c#
 while (true) {
     try {
         Thread.Sleep(3);
         contex.Update();
         server.Update();
     }
```

Server.cs，定时，Server会在到达一定时间之后，尝试去做DoUpdate

```c#
        public void   Update(){
            var now = DateTime.Now;
            _deltaTime = (now - _lastUpdateTimeStamp).TotalSeconds;
            if (_deltaTime > UpdateInterval) {
                _lastUpdateTimeStamp = now;
                _timeSinceStartUp = (now - _startUpTimeStamp).TotalSeconds;
                DoUpdate();
            }
        }
        public void DoUpdate(){
            //check frame inputs
            var fDeltaTime = (float) _deltaTime;
            var fTimeSinceStartUp = (float) _timeSinceStartUp;
            _room?.DoUpdate(fTimeSinceStartUp, fDeltaTime);
        }
```

而在Room当中的DoUpdate会尝试取到所有的客户端发来的输入消息，然后广播

```c#
//Room.cs
        public void DoUpdate(float timeSinceStartUp, float deltaTime){
            if (!IsRunning) return;
            CheckInput();
        }


        private void CheckInput(){
            if (_tick2Inputs.TryGetValue(_curTick, out var inputs)) {
                if (inputs != null) {
                    bool isFullInput = true;
                    for (int i = 0; i < inputs.Length; i++) {
                        if (inputs[i] == null) {
                            isFullInput = false;
                            break;
                        }
                    }

                    if (isFullInput) { //<--这里会有一个问题，就是如果客户端没有全部发来消息，如何处理（通过预测+回滚）
                        BoardInputMsg(_curTick, inputs);
                        _tick2Inputs.Remove(_curTick);
                        _curTick++;
                    }
                }
            }
        }
```

我们这里要先关心，_tick2Inputs这个存储所有用户输入的Dictionary是如何更新的`private Dictionary<int, PlayerInput[]> _tick2Inputs = new Dictionary<int, PlayerInput[]>();`

注意这个dic的Key是tick，也就是当前帧，value是当前帧，所有用户传过来的输入（这里的问题是，如何保证客户端和服务器帧同步）

他是在这个函数回调：

```c#
//Room.cs
        public void OnPlayerInput(int useId, Msg_PlayerInput msg){
            //Debug.Log($" Recv Input: {useId} {msg.input.inputUV}");
            int localId = 0;
            if (!_id2LocalId.TryGetValue(useId, out localId)) return;
            PlayerInput[] inputs;
            if (!_tick2Inputs.TryGetValue(msg.tick, out inputs)) {
                inputs = new PlayerInput[MaxPlayerCount];
                _tick2Inputs.Add(msg.tick, inputs);
            }

            inputs[localId] = msg.input;
            CheckInput();
        }

//而这个OnPlayerInput最终是在Dispatch当中调用的：
//Server.cs
void OnPlayerInput(Session session, IMessage message){
            var msg = message as Msg_PlayerInput;
            var player = session.GetBindInfo<PlayerServerInfo>();
            _room?.OnPlayerInput(player.Id, msg);
        }
public void Dispatch(Session session, Packet packet){
    ushort opcode = packet.Opcode();
    var message = session.Network.MessagePacker.DeserializeFrom(opcode, packet.Bytes, Packet.Index,
        packet.Length - Packet.Index) as IMessage;
    //var msg = JsonUtil.ToJson(message);
    //Log.sLog("Server " + msg);
    var type = (EMsgType) opcode;
    switch (type) {
        case EMsgType.JoinRoom:
            OnPlayerConnect(session, message);
            break;
        case EMsgType.QuitRoom:
            OnPlayerQuit(session, message);
            break;
        case EMsgType.PlayerInput:
            OnPlayerInput(session, message);
            break;
        case EMsgType.HashCode:
            OnPlayerHashCode(session, message);
            break;
    }
}
    
```

那么这个Server的Dispatch是在哪里调用的呢？

其实是在上面建立的TChannel，再通过TChannel建立的Session那边调用的：

```c#
//Session

	
	   public void Start(){
            this.StartRecv();
        }
	   private async void StartRecv(){
            while (true) {
                if (this.IsDisposed) {
                    return;
                }

                Packet packet;
                try {
                    packet = await this.channel.Recv(); //<--如果客户端有东西发过来

                    if (this.IsDisposed) {
                        return;
                    }
                }
                catch (Exception e) {
                    Log.Error(e.ToString());
                    continue;
                }

                try {
                    this.Run(packet); //<--就调用Run
                }
                catch (Exception e) {
                    Log.Error(e.ToString());
                }
            }
        }
        private void Run(Packet packet){
            if (packet.Length < Packet.MinSize) {
                Log.Error($"message error length < {Packet.MinSize}, ip: {this.RemoteAddress}");
                this.Network.Remove(this.Id);
                return;
            }

            byte flag = packet.Flag();
            ushort opcode = packet.Opcode();
            if (OpcodeHelper.IsClientHotfixMessage(opcode)) {
                this.Network.MessageDispatcher.Dispatch(this, packet); //<--这里就是Server，因为Server实现了IMessageDispatcher接口
                return;
            }
        //...
        }
```

这个调用链是：

Server在Awake的时候，在StartAccept开启了轮询，接受客户端链接，每有一个客户端完成链接，就会建立起一个TChannel，并且建立一个Session

而后，session的Start函数会被调用，这个函数会调用到StartRecv，而StartRecv会不断的检查来自客户端的发送，如果客户端有东西发过来，就调用Run，最后调用到Server的Dispatch当中，并且根据发送过来消息的类型，调用对应的回调函数。

比如如果是输入调用，当检查完所有的客户端都发过来了，那就广播，并且服务器定时也会检查输入（也就是在Room.DoUpdate当中）