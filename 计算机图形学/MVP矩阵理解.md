# MVP矩阵理解

每个模型都有自己的模型坐标系

把模型坐标系转换到世界坐标系，这个过程就是model变换

把世界坐标系转换到camera坐标系，那么就是view变化

把view坐标系下的vertex位置，转换到在二维近平面上的投影坐标，这个过程就是projection矩阵。（projection矩阵是矩阵，只做线性变换，投影除法是无法用projection矩阵来完成的（属于非线性变换））

