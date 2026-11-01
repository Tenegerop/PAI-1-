cd && git clone --recursive https://github.com/matsuolab/pai_training　　　     #Lerobot環境構築
cd pai_training

curl -fsSL https://pixi.sh/install.sh | bash
echo 'eval "$(pixi completion --shell bash)"' >> ~/.bashrc
source ~/.bashrc

pixi install


mkdir -p ~/pai_ws/src　　#テレオペ導入
cd ~/pai_ws/src && git clone https://github.com/matsuolab/physicalai_teleop.git
cd ~/pai_ws && rosdep install --from-paths src --ignore-src -r -y 
colcon build --symlink-install --packages-select physicalai_teleop 

source install/setup.bash 


ros2 launch crane_plus_gazebo crane_plus_with_table.launch.py　#gazebo CRANE +V2起動


source install/setup.bash  　　　　　　　　　　　　　　　　#テレオペ起動
ros2 run physicalai_teleop teleop_keyboard 

Teleop Crane+ V2 (Direct Joint Control):
---------------------------------------
Q/E: Select joint (1-4)
W/S: + / - selected joint
R/F: + / - all joints (coarse)
O:   Open gripper
C:   Close gripper
X:   Reset robot to vertical (all-zero) position
ESC: Quit

Selected joint: crane_plus_joint1

ros2 bag record -a -o ~/dataset/rosbags/pekori/ep0　　#今回はep10までデータ取得

cd ~/pai_training　#データ加工
pixi run convert

pixi run train　#学習実行

pixi run infer --model-path /home/ubuntu/checkpoints/train/crane_plus_pekori_act　　#デプロイ



