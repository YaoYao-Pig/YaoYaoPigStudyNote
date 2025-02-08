# 项目经历：

Dungeon Guner

项目介绍：类挺进地牢的2.5D RougeLike射击游戏

技术总结：

1. 基于ScriptableObject进行地牢地图模板、玩家信息、敌人信息等管理，可以通过简单的配置实现快速的新增人物或者敌人等。
2. 游戏的地牢关卡基于Tilemap，其随机生成模板也通过ScriptableObject进行管理，使用EditorWindow实现了一个可视化的地牢关卡的可视化创建工具。
3. 构建了基于事件驱动的UI系统，射击系统，敌人AI系统。以AI系统为例，
4. 使用AStart算法实现了基于Tilemap的Grid的寻路。并且利用Unity的Profile工具，通过观察CPU占用优化寻路算法的实现，提高寻路性能表现。
5. 游戏中存在大量的比如子弹，特效，音效等需要频繁创建销毁的GameObject，通过通用对象池管理上述对象，进行优化。