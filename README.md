

# Characterizing the User Interests on YouTube

Social media has become a part to our daily routine, through multiple social platforms. This work focuses on the YouTube data by creating an embedding space at the video level.  We use the Latent Dirichlet Allocation (LDA) algorithm to perform topic modelling, which resulted to  be  much  more  sensitive  to  the  input  data  than  to  the  model  parameters.   Furthermore, we  implemented  different  methods  to  evaluate  and  interpret  the  results.   Finally,  a  human evaluation task is performed on our optimal models in order to verify the performance of our evaluation metrics.  We hope that our work can help future projects which focus on performing embeddings on very large dataset.

Table of Contents
=================
  * [Full Documentation](https://github.com/epfl-dlab/youtube_embeddings/blob/Olivier/project_report.pdf)
  * [Datasets](#datasets)
  * [Quickstart](#quickstart)
  * [Author](#author)
  

## Datasets

The table below summaries the dataset used during the project.

| Dataset        | Description    |  
| --------   | -----:   | 
| yt_metadata_en.jsonl.gz        | The dataset contains the features of 73’301’516 YouTube videos from 24/05/2005 to 20/11/2019. | 
| df_channels_en.tsv.gz        | The dataset contains the features of 136’470 YouTube channels.   |  



* yt_metadata_en: This dataset contains the following features for each video : `categories` `channel_id` `crawl_date` `description` `dislike_count` `like_count` `display_id` `duration` `tags`  `title`  `upload_date`  `view_count`  

* df_channels_en: This dataset contains the following features for each channel : `category` `join_date` `channel_id` `name` `subscribers_counts` `videos_counts` `subscriber_rank_sb` `weights`



## Quickstart

### Step 1: Generate the data for topic modelling

Users can find the  source codes in the folder `src`. 

**Now, we can generate the correct data for topic modelling by executing:**

	$ python src/data_processing.py [--path_dataset <String>] [--path_write_data <String>] [--n_min_sub <int>] [--n_min_views <int>] [--use_bigram] [--min_vid_per_token <int>] [--n_top_vid_per_combination <int>] [--n_jobs <int>] [--executor_mem <int>] [--driver_mem <int>]

where parameters in [ ] are optional. 

- `--path_dataset <String>`: Specify the path to the dataset. We let this parameter in order to apply the same work on other dataset.

- `--path_write_data <String>`: Specify the path to the directory where the user wants to save all the intermediate results. *Default value: /dlabdata1/youtube_large/olam/data/test/*

- `--n_min_sub <int>`: Specify the threshold for the minimum number of subscrbers for relevant channels. *Default value: 100000*

- `--n_min_views <int>`: Specify the threshold for the minimum number of views for relevant videos. *Default value: 10000*

- `--use_bigram`: Specify if the user want to generate bi-grams for the vocabulary. 

- `--min_vid_per_token <int>`: Specify the threshold *t* for tokens that appear in at least *t* videos to be considered as relevant tokens. *Default value: 100*

- `--n_top_vid_per_combination <int>`: Specify the threshold for selecting the number videos with the most views for each combination of `category`, `uploaded_year` and `channel_id` for topic modelling. *Default value: 20*

- `--n_jobs <int>`: Specify the number of jobs for creating the data for topic modelling. *Default value: 4*

- `--executor_mem <int>`: Specify the memory in g for each executor. *Default value: 8*

- `--driver_mem <int>`: Specify the memory in g for the driver. *Default value: 64*

**Examples:**

	$ python data_processing.py --use_bigram --min_vid_per_token 20

The output files are saved in the `/dlabdata1/youtube_large/olam/data/test/` directory, where bi-grams will be generated in the vocabulary and the threshold *t* for tokens that appear in at least *t* videos to be considered as relevant tokens is 20.

### Step 2: Training the topic model with LDA


#### Please note the following and  follow the steps below in order to train the LDA model:

Define the following variables :
- `input_data` : the path of the `sparkdf.json` file generated by step 1.
- `path_dir`: the path of the directory where the user want to store the intermediate results.
- `path_hdfs` : the path of the directory in `hdfs`. This is accessible if the user have access to the Hadoop cluster. The main `hdfs`commands are shown and explained below

First of all, the user need to store the data in the Hadoop cluster. Then, the user need to create the followings folders in `path_hdfs`: `describe_topics` `topics_doc_matrix` `models`.

**Commands in Hadoop cluster :**

	$ hdfs dfs -ls path_hdfs ### check the files in the directory `path_hdfs`
	$ hdfs dfs -mkdir path_new_dir ### Create a directory in hadoop
	$ hdfs dfs -put localfile path_hdfs ### Copy the local file to the hadoop directory `path_hdfs`
	$ hdfs dfs -get hdfs_file path_dir ### Copy the file in the hadoop directory to `path_file`


**Now, the user can train the LDA models by using the following command:**

	$ spark-submit --master yarn --deploy-mode cluster --num-executors 47 --executor-cores 4 --driver-memory 64g --executor-memory 4g src/train_LDA.py [--min_n_topic <int>] [--max_n_topic <int>] [--tune] [--n_topic_tune <int>] [--path_data <String>] [--n_iter <int>]

The output files are saved in the `--path_data` directory.

- `--min_n_topic <int>`: Specify the minimum number of topics for the LDA model. Multiples models will be computer from `the min_n_topic` `to max_n_topic`, with a step of 5. *Default value: None*

- `--max_n_topic <int>`: Specify the maximum number of topics for the LDA model. Multiples models will be computer from `the min_n_topic` `to max_n_topic`, with a step of 5. *Default value: None*

- `--tune`:  Boolean that indicate that computing models with specific values for document concentration and topic concentration will be performed. Required n_topic_tune. *Default value: False*

- `--n_topic_tune <int>`: Specify the number of topics of the models on which pecific values for document concentration and topic concentration will be tuned. *Default value: None*

- `--path_data <String>`: Specify the path of the directory where the results will be saved. *Default value: /user/olam/test/*

- `--n_iter <int>`: Specify the number of iterations for the LDA algorithm. *Default value: 500*

**Example:**

	$ spark-submit --master yarn --deploy-mode cluster --num-executors 47 --executor-cores 4 --driver-memory 64g --executor-memory 4g train_LDA.py --min_n_topic 30 --max_n_topic 45

The above commands will compute the LDA models with 30, 35, 40 and 45 topics. All the results will be stored in the default `path_data` 

An important note is that the parameters of the `spark-submit` command are optimal to our task. The user has to make sure that if he runs the script on other data, he may need to change the `num-executors`, `executor-cores`, `driver-memory` and `executor-memory` parameters.

Once the run is done, make sure to copy the resulted files from Hadoop to the local directory. In particular, the files in `path_hdfs/describe_topics/` should be in `path_dir/describe_topics/` where `path_dir` should be the same as `path_write_data` from step 1. This should be done for the following directories : `describe_topics`, `topics_doc_matrix` `models`.

### Step 3: Compute Topic Coherences


**In order to compute the topic coherences of the models, run the following command:**

	$ python src/topic_coherence.py [--min_n_topic <int>] [--max_n_topic <int>] [--n_topics <int>] [--tune_alpha_beta] [--path_data <String>] [--n_jobs <int>]

This will generate a line plot if `--tune_alpha_beta` is not used and a heat map for the opposite case, where the figures will represent the *c_v* and  *u_mass* coherence score for the selected models.

- `--min_n_topic <int>`: Specify the number of topics from which the topic coherences will be computed. Multiples models will be computer from `the min_n_topic` `to max_n_topic`, with a step of 5. *Default value: None*

- `--max_n_topic <int>`: Specify the number of topics until which the topic coherences will be computed. Multiples models will be computer from `the min_n_topic` `to max_n_topic`, with a step of 5. *Default value: None*

- `--tune_alpha_beta`:  Boolean that indicate that computing coherence scores for models with specific values for document concentration and topic concentration will be performed. Required n_topic_tune. *Default value: False*

- `--n_topic_tune <int>`: Specify the number of topics of the models where the coherence scores will be computed on specific values for document concentration and topic concentration. *Default value: None*

- `--path_data <String>`: Specify the path of the directory where input files can be found. It should be the same as `path_write_data` from step 1. *Default value: /dlabdata1/youtube_large/olam/data/test/*

- `--n_iter <int>`: Specify the number of iterations for the LDA algorithm. *Default value: 500*

### Step 4: Build a classifier to check the performance of the topic models

**In order to build a classifier to check the performance of the topic models, the user should run the following command:**

	$ python src/cat_classifier.py [--path_dataset <String>] [--path_write_data <String>] [--n_min_sub_classifier <int>] [--n_min_views_classifier <int>] [--n_top_vid_per_combination <int>] [--min_n_topic <int>] [--max_n_topic <int>] [--tune] [--n_topic_tune <int>] [--n_jobs <int>] [--executor_mem <int>] [--driver_mem <int>]   

This will generate a line plot if `--tune` is not used and a heat map for the opposite case, where the figures will represent the accuracy score of the classifier for the selected models.


- `--path_dataset <String>`: Specify the path to the dataset. Should be the same dataset that is used in step 1. 

- `--path_write_data <String>`: Specify the path to the directory where the user wants to save all the intermediate results. Should be the same `path_write_data` that is used in step 1. *Default value: /dlabdata1/youtube_large/olam/data/test/*

- `--n_min_sub_classifier <int>`: Specify the threshold for the minimum number of subscrbers for relevant channels of videos in the classification task. *Default value: None*

- `--n_min_views_classifier <int>`: Specify the threshold for the minimum number of views for relevant videos in the classification task. *Default value: 10000*

- `--n_top_vid_per_combination <int>`: Specify the threshold for selecting the number videos with the most views for each combination of `category`, `uploaded_year` and `channel_id` for topic modelling. Must be the same as `n_top_vid_per_combination` used in step 1. *Default value: 20*

- `--min_n_topic <int>`: Specify the minimum number of topics for the LDA model. One classifier per model will be created and accuracies for each classifier are compared. *Default value: None*

- `--max_n_topic <int>`: Specify the maximum number of topics for the LDA model. One classifier per model will be created and accuracies for each classifier are compared. *Default value: None*

- `--tune`: Boolean that indicate that tuning the document concentration and topic concentration will be performed. Required n_topic_tune. *Default value: False*

- `--n_topic_tune <int>`: Specify the number of topics on which document concentration and topic concentration will be tuned. *Default value: None*

- `--n_jobs <int>`: Specify the number of jobs for transforming the data for the classifier from the LDA model. *Default value: 4*

- `--executor_mem <int>`: Specify the memory in g for each executor. *Default value: 4*

- `--driver_mem <int>`: Specify the memory in g for the driver. *Default value: 64*

### Step 5: Get the visualisation of the topic models

In order to get a nice representation of the LDA model, the user can run the following notebook : `notebook/visualisation_lda.ipynb`

The visualisation of our model with 55 topics can be found [here](https://htmlpreview.github.io/?https://github.com/olivierlam97/EPFL-SemesterProject-YouTube/blob/master/html/model55.html)

The visualisation of our model with 110 topics can be found [here](https://htmlpreview.github.io/?https://github.com/olivierlam97/EPFL-SemesterProject-YouTube/blob/master/html/model110.html)



## Author

Olivier Quốc-Vinh Lam
