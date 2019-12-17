## Data
1. rgb_detection_train.txt

        # type = {1:'Pedestrian', 2:'Car', 3:'Cyclist'}
        img_path typeid confidence xmin ymin xmax ymax
        dataset/KITTI/object/training/image_2/000000.png 1 0.999559 718 141 807 311

2. frustum_carpedcyc_train.pickle

        pickle.dump(id_list, fp)
        pickle.dump(box2d_list,fp)
        pickle.dump(input_list, fp)
        pickle.dump(type_list, fp)
        pickle.dump(frustum_angle_list, fp)
        pickle.dump(prob_list, fp)

3. label_2(kitti/kitti_util.py::Object3d)

        Car 0.00 0 1.85 387.63 181.54 423.81 203.12 1.67 1.87 3.69 -16.53 2.39 58.49 1.57

        self.type = data[0] # 'Car', 'Pedestrian', ...
        self.truncation = data[1] # truncated pixel ratio [0..1]
        self.occlusion = int(data[2]) # 0=visible, 1=partly occluded, 2=fully occluded, 3=unknown
        self.alpha = data[3] # object observation angle [-pi..pi]

        # extract 2d bounding box in 0-based coordinates
        self.xmin = data[4] # left
        self.ymin = data[5] # top
        self.xmax = data[6] # right
        self.ymax = data[7] # bottom
        self.box2d = np.array([self.xmin,self.ymin,self.xmax,self.ymax])
        
        # extract 3d bounding box information
        self.h = data[8] # box height
        self.w = data[9] # box width
        self.l = data[10] # box length (in meters)
        self.t = (data[11],data[12],data[13]) # location (x,y,z) in camera coord.
        self.ry = data[14] # yaw angle (around Y-axis in camera coordinates) [-pi..pi]

4. calib(kitti/kitty_util.py::Calibration)

        P0: 7.070493000000e+02 0.000000000000e+00 6.040814000000e+02 0.000000000000e+00 0.000000000000e+00 7.070493000000e+02 1.805066000000e+02 0.000000000000e+00 0.000000000000e+00 0.000000000000e+00 1.000000000000e+00 0.000000000000e+00
        P1: 7.070493000000e+02 0.000000000000e+00 6.040814000000e+02 -3.797842000000e+02 0.000000000000e+00 7.070493000000e+02 1.805066000000e+02 0.000000000000e+00 0.000000000000e+00 0.000000000000e+00 1.000000000000e+00 0.000000000000e+00
        P2: 7.070493000000e+02 0.000000000000e+00 6.040814000000e+02 4.575831000000e+01 0.000000000000e+00 7.070493000000e+02 1.805066000000e+02 -3.454157000000e-01 0.000000000000e+00 0.000000000000e+00 1.000000000000e+00 4.981016000000e-03
        P3: 7.070493000000e+02 0.000000000000e+00 6.040814000000e+02 -3.341081000000e+02 0.000000000000e+00 7.070493000000e+02 1.805066000000e+02 2.330660000000e+00 0.000000000000e+00 0.000000000000e+00 1.000000000000e+00 3.201153000000e-03
        R0_rect: 9.999128000000e-01 1.009263000000e-02 -8.511932000000e-03 -1.012729000000e-02 9.999406000000e-01 -4.037671000000e-03 8.470675000000e-03 4.123522000000e-03 9.999556000000e-01
        Tr_velo_to_cam: 6.927964000000e-03 -9.999722000000e-01 -2.757829000000e-03 -2.457729000000e-02 -1.162982000000e-03 2.749836000000e-03 -9.999955000000e-01 -6.127237000000e-02 9.999753000000e-01 6.931141000000e-03 -1.143899000000e-03 -3.321029000000e-01
        Tr_imu_to_velo: 9.999976000000e-01 7.553071000000e-04 -2.035826000000e-03 -8.086759000000e-01 -7.854027000000e-04 9.998898000000e-01 -1.482298000000e-02 3.195559000000e-01 2.024406000000e-03 1.482454000000e-02 9.998881000000e-01 -7.997231000000e-01


## Step

1. dataset 준비 

        ln -s /data1/dataset/kitti KITTI
       
2. dataset 조회

        python kitti/kitti_object.py
        
3. prepare pickle

        python kitti/prepare_data.py --gen_train --gen_val --gen_val_rgb_detection
        python kitti/prepare_data.py --demo

4. train

        CUDA_VISIBLE_DEVICES=0 python train/test.py \
        --gpu 0 \
        --num_point 1024 \
        --model frustum_pointnets_v1 \
        --model_path train/log_v1/model.ckpt \
        --output train/detection_results_v1 \
        --data_path kitti/frustum_carpedcyc_val_rgb_detection.pickle \
        --from_rgb_detection \
        --idx_path kitti/image_sets/val.txt \
        --from_rgb_detection
        
        train/kitti_eval/evaluate_object_3d_offline dataset/KITTI/object/training/label_2/ train/detection_results_v1

5. evaluate

        python train/test.py \
        --gpu 0 \
        --num_point 1024 \
        --model frustum_pointnets_v1 \
        --model_path train/log_v1/model.ckpt \
        --output train/detection_results_v1 \
        --data_path kitti/frustum_carpedcyc_val_rgb_detection.pickle \
        --from_rgb_detection \
        --idx_path kitti/image_sets/val.txt \
        --from_rgb_detection
        
        train/kitti_eval/evaluate_object_3d_offline dataset/KITTI/object/training/label_2/ train/detection_results_v1
