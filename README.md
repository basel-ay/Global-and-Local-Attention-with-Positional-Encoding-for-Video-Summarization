# PGL-SUM: Combining Global and Local Attention with Positional Encoding for Video Summarization

## PyTorch Implementation of PGL-SUM
- From **"PGL-SUM: Combining Global and Local Attention with Positional Encoding for Video Summarization"** (submitted for publication at the IEEE ISM 2021 Conference)
- Written by Evlampios Apostolidis, Georgios Mpalaouras, Vasileios Mezaris and Ioannis Patras
- This software can be used for training a deep learning architecture which estimates frames' importance after modeling their dependencies with the help of global and local multi-head attention mechanisms that integrate a positional encoding component. Training is performed in a supervised manner based on ground-truth data (human-generated video summaries). After being trained on a collection of videos, the PGL-SUM model is capable of producing representative summaries for unseen videos, according to a user-specified time-budget about the summary duration.

## Main dependencies
- Python  3.6
- PyTorch 1.0.1

## Data
Structured h5 files with the video features and annotations of the SumMe and TVSum datasets are available within the "data" folder. The GoogleNet features of the video frames were extracted by [Ke Zhang](https://github.com/kezhang-cs) and [Wei-Lun Chao](https://github.com/pujols) and the h5 files were obtained from [Kaiyang Zhou](https://github.com/KaiyangZhou/pytorch-vsumm-reinforce). These files have the following structure:
<pre>
/key
    /features                 2D-array with shape (n_steps, feature-dimension)
    /gtscore                  1D-array with shape (n_steps), stores ground truth improtance score (used for training, e.g. regression loss)
    /user_summary             2D-array with shape (num_users, n_frames), each row is a binary vector (used for test)
    /change_points            2D-array with shape (num_segments, 2), each row stores indices of a segment
    /n_frame_per_seg          1D-array with shape (num_segments), indicates number of frames in each segment
    /n_frames                 number of frames in original video
    /picks                    positions of subsampled frames in original video
    /n_steps                  number of subsampled frames
    /gtsummary                1D-array with shape (n_steps), ground truth summary provided by user (used for training, e.g. maximum likelihood)
    /video_name (optional)    original video name, only available for SumMe dataset
</pre>
Original videos and annotations for each dataset are also available in the authors' project webpages:
- TVSum dataset: https://github.com/yalesong/tvsum
- SumMe dataset: https://gyglim.github.io/me/vsum/index.html#benchmark

## Training
To train the model using one of the aforementioned datasets and for a number of randomly created splits of the dataset (where in each split 80% of the data is used for training and 20% for testing) use the corresponding JSON file that is included in the "data/splits" directory. This file contains the 5 randomly-generated splits that were utilized in our experiments.

For training the model using a single split, run:
<pre>
python main.py --split_index N --n_epochs E --batch_size B --video_type 'dataset_name'
</pre>
In the command above: N refers to index of the used data split, E refers to the number of training epochs (default value: 200), B refers to the batch size (default value for training in full-batch mode: 20 for SumMe, 40 for TVSum), and 'dataset_name' refers to the name of the used dataset (can be either 'SumMe' or 'TVSum').

Alternatively, to train the model for all 5 splits, use the ['run_summe_splits.sh'](https://github.com/e-apostolidis/PGL-SUM/blob/master/model/run_summe_splits.sh) and/or ['run_tvsum_splits.sh'](https://github.com/e-apostolidis/PGL-SUM/blob/master/model/run_tvsum_splits.sh) script and do the following:
<pre>
chmod +x run_summe_splits.sh    # Makes the script executable.
chmod +x run_tvsum_splits.sh    # Makes the script executable.
./run_summe_splits              # Runs the script. 
./run_tvsum_splits              # Runs the script.  
</pre>
Please note that after each training epoch the algorithm performs an evaluation step, using the trained model to compute the importance scores for the frames of each video of the test set. These scores are then used by the provided evaluation scripts to assess the overal performance of the model (in F-Score).

The progress of the training can be monitored via the TensorBoard platform and by:
- opening a command line (cmd) and running: tensorboard --logdir=/path/to/log-directory --host=localhost
- opening a browser and pasting the returned URL from cmd

## Configurations
Setup for the training process:

- In ['data_loader.py'](https://github.com/e-apostolidis/PGL-SUM/blob/master/model/data_loader.py), specify the path to the 'h5' file of the used dataset and the path to the 'json' file containing data about the utilized data splits.
- In ['configs.py'](https://github.com/e-apostolidis/PGL-SUM/blob/master/model/configs.py), define the directory where the analysis results will be saved to.
    
Arguments in ['configs.py'](https://github.com/e-apostolidis/PGL-SUM/blob/master/model/configs.py): 
<pre>
--video_type: The used dataset for training the model. Can be either 'SumMe' or 'TVSum'.
--input_size: The size of the input feature vectors (default value: 1024 for GoogLeNet features).
--seed: The number of the utilized seed for generating random weights during the network's initialization (default value: 12345).
--fusion: The type of the used approach for feature fusion (default option: "add"; other options: "mult", "avg" and "max").
--n_segments: The number of video segments that is equal to the number of local attention mechanisms (default value: 4).
--pos_enc: The type of the applied positional encoding (default option: "absolute"; other options: "relative" and "none").
--heads: The number of heads of the global attention mechanism (default value: 4).
--n_epochs: The number of training epochs (default value: 200).
--batch_size: The size of the training batch (default value: 20 for SumMe and 40 for TVSum).
--clip: The gradient clipping parameter (default value: 5).
--lr: The value of the adopted learning rate (default value: 5e-5).
--l2_req: The value of the regularization factor (default value: 1e-5)
--split_index: The index of the utilized data split (ranging between 0 and 4 in our experiments).
--init_type: The adopted weight initialization method (default option: 'xavier'; other options: 'normal', 'kaiming' and 'orthogonal')
</pre>

## Model Selection and Evaluation 
The utilized model selection criterion relies on the post-processing of the calculated losses over the training epochs and enables the selection of a well-trained model by indicating the training epoch. To evaluate the trained models of the architecture and automatically select a well-trained model, run ['evaluate_exp.sh'](https://github.com/e-apostolidis/PGL-SUM/blob/master/evaluation/evaluate_exp.sh). To run this file, specify: i) the path to the folder where the analysis results are stored, ii) the dataset, and iii) the used approach for computing the overal F-Score after comparing the generated summary with all the available user summaries (i.e., 'max' for SumMe and 'avg' for TVSum). For further details about the adopted structure of directories in our implementation, please check line [#6](https://github.com/e-apostolidis/PGL-SUM/blob/master/evaluation/evaluate_exp.sh#L6) and line [#11](https://github.com/e-apostolidis/PGL-SUM/blob/master/evaluation/evaluate_exp.sh#L11) of ['evaluate_exp.sh'](https://github.com/e-apostolidis/PGL-SUM/blob/master/evaluation/evaluate_exp.sh).

## License
Copyright (c) 2021, Evlampios Apostolidis, Georgios Mpalaouras, Vasileios Mezaris, Ioannis Patras / CERTH-ITI. All rights reserved. This code is provided for academic, non-commercial use only. Redistribution and use in source and binary forms, with or without modification, are permitted for academic non-commercial use provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.
2. Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation provided with the distribution.

This software is provided by the authors "as is" and any express or implied warranties, including, but not limited to, the implied warranties of merchantability and fitness for a particular purpose are disclaimed. In no event shall the authors be liable for any direct, indirect, incidental, special, exemplary, or consequential damages (including, but not limited to, procurement of substitute goods or services; loss of use, data, or profits; or business interruption) however caused and on any theory of liability, whether in contract, strict liability, or tort (including negligence or otherwise) arising in any way out of the use of this software, even if advised of the possibility of such damage.

## Acknowledgement
This work was supported by the EU Horizon 2020 programme under grant agreement H2020-832921 MIRROR, and by EPSRC under grant No. EP/R026424/1.