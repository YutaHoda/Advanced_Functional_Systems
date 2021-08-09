# Advanced_Functional_Systems

## Operation Environment
Ubuntu 18.04
ROS melodic

## Install & Build
```
mkdir -p catkin_ws/src
cd ~/catkin_ws/src
git clone git@github.com:YutaHoda/Advanced_Functional_Systems.git

cd .. 
catkin_make -DCMAKE_BUILD_TYPE=Release
```

## How to use
インストール時に，catkin_ws以外にクローンした場合はそのワークスペースにbashを通してから実行する必要がある．

### 1 MoveIt!を使ったマニピュレータの制御
#### 1.1 ROS/MoveIt!によるマニピュレータの制御
使用するマニピュレータのモデルをRvizで確認する．下記コマンドを実行し，マニピュレータモデルの操作GUI及びRvizを起動する．
```
$ roslaunch sixdofarm sixdofarm.launch
$ rosrun rviz rviz
```
Rviz起動後，「Add」ボタンより「RobotModel」を追加し，「Global Options」のFixed Frameをbaseに変更する．これによりRvizにマニピュレータモデルの表示が可能となる．また，一つ目のコマンドで表示されたウィンドウを操作することでマニピュレータの各関節が回転することが確認できる．

MoveItで作成した自作モデルを用いたマニピュレータの制御を行う．以下のコマンドを実行し，自作したマニピュレータがRvizに表示されることを確認する．
```
$ roslaunch sixdofarm_moveit_config demo.launch
```
マニピュレータの先端にあるボール状の部分をドラッグすることでマニピュレータの姿勢が変化することが確認できる．
Rvizの「Planning」タブにある「plan」ボタンを押すことで直立している姿勢から変化させた姿勢までの軌道を計画する．そして，「Execute」ボタンにより実行できる．

#### 1.2 Gazeboによるマニピュレータのシミュレーション
マニピュレータの動作シミュレーションをGazeboで行う．以下のコマンドを実行し，マニピュレータモデル操作GUI，Gazebo及びRvizを起動する．なお，処理負荷の関係上，Gazeboにガソリンスタンドは表示していない．
```
$ roslaunch sixdofarm_moveit_config sixdofarm_moveit.launch
```
Rvizでマニピュレータの制御を確認した時と同様に，マニピュレータの先端にあるボール状の部分をドラッグして，マニピュレータ目標位置を指定する．そして，「Plan」ボタンを押してMoveItに計画してもらう．次に，「Execute」ボタンにより計画された軌道をGazebo上のマニピュレータへ指示する．すると，Rviz上で指示されたマニピュレータの計画軌跡通りにGazebo上のマニピュレータが動作することが確認できる．

### 2 ROSを使った移動ロボットの制御御
#### 2.1 Gazebo上での移動ロボットモデルの構築
移動モデルをRvizとGazeboで動作させる．下記コマンドを実行し，RvizとGazeboを起動する． 
```
$ roslaunch wheel_robot wheel_robot_control.launch
```
rqtを起動し，「Plagins」→「Robot Tools」→「Robot Steering」ツールを立ち上げる．これによりGUIで移動ロボットが動くことが確認できる．
また，下記コマンドによりキーボード操作で動かすことも可能である．
```
$ rosrun wheel_robot_control key_teleop.py
```

#### 2.2 移動ロボットのナビゲーション
まず，走行軌跡の保存を行います．下記コマンドを実行し，シミュレータ，走行軌跡保存ノード，キーボード操作ノードを起動する．
```
$ roslaunch wheel_robot wheel_robot_control.launch
$ rosrun odom_listener odom_listener
$ rosrun wheel_robot_control key_teleop.py
```
キーボード操作によって移動した通りの走行軌跡が保存される．なお，走行軌跡のwaypoints.yamlは走行軌跡保存ノードを起動したディレクトリに保存される．

次に保存した走行軌跡に移動ロボットを追従させる．以下のコマンドを起動することで，RvizとGazeboを立ち上げ，waypointsに従って移動する．
```
$ roslaunch wheel_robot wheel_robot_control.launch
$ roslaunch wheel_robot_nav wheel_robot_nav.launch
```
また，Rvizの「Add」ボタンより「By topic」タブの「visualization marker」を追加することでwaypointsの表示を，「current_goal」を追加することで現在の目標ゴール位置が確認できる．さらに，「path」を追加することでゴールまでの経路計画が確認できる．

#### 2.3 移動ロボットで地図を作成
Gazebo上にオブジェクトを配置して，シミュレータの地図を作成する．以下のコマンドを実行して，Gazeboとgmapping，キーボード操作ノード，rvizの起動を行う．なお，Gazeboの起動後，gmappingを起動する前にシミュレータ上にオブジェクトを配置する．
```
$ roslaunch wheel_robot wheel_robot.launch
$ roslaunch wheel_robot_gmapping wheel_robot_gmapping.launch
$ rosrun wheel_robot_control key_teleop.py
$ rosrun rviz rviz
```
地図の作成状況を確認するため，Rvizに「Add」ボタンから「RobotModel」と「Map」を追加する．そして，キーボード操作により移動ロボットを動かして地図作製を行う．
地図作製が完了したら，以下コマンドで地図を保存する．なお，testは任意のファイル名である．
```
$ rosrun map_server map_saver -f <test>
```
地図はコマンドを実行したディレクトリに保存される．地図はpgmファイルとyamlファイルで保存される．

#### 2.5 差動2輪移動ロボットのナビゲーション
差動2輪移動ロボットのナビゲーションを行うため，地図作製を行う．以下のコマンドを実行して，Gazebo，gmapping，キーボード操作ノードを起動する．今回はWillow Garageの地図を作成する．なお，Rvizも起動してmap表示をすると地図更新の様子が確認できる．
```
$ roslaunch diff_mobile_robot diff_mobile_gazebo.launch
$ roslaunch diff_mobile_robot gmapping.launch
$ roslaunch diff_mobile_robot keyboard_teleop.launch
```

次にナビゲーションをするためのwaypointsを作成する．4輪モデルの時と同様に以下のコマンドを実行して，Gazebo，amcl，走行軌跡保存ノード，キーボード操作ノードを起動する．

```
$ roslaunch diff_mobile_robot diff_mobile_gazebo.launch
$ roslaunch diff_mobile_robot amcl.launch
$ rosrun odom_listener odom_listener
$ rosrun wheel_robot_control key_teleop.py
```

そして，保存した走行軌跡を用いて移動ロボットのナビゲーションを行う．以下のコマンドを実行して，Gazebo，amcl，ナビゲーションノードを起動する．
```
$ roslaunch diff_mobile_robot diff_mobile_gazebo.launch
$ roslaunch diff_mobile_robot amcl.launch
$ roslaunch diff_mobile_robot wheel_robot_nav.launch
```
なお，Rvizを用いることでwaypointsや目標経路の確認ができる．以上により移動ロボットのナビゲーションが可能となる．