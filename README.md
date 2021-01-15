## Characterizing the user interests
### Channel embedding
    
    
Based on the YouTube comments from 2005 to 2019 this project consists of characterizing the evolution of user interest. In order to find and analyse the communities in this huge social platform, we have to perform an embedding at the video level and an embedding at the channel level. Naturally, the two embeddings should share common characteristics which would determine the behaviour of users on YouTube. This work done on this project focuses on the channel embedding while a parallel work focuses on the video embedding. Results are expected to be shared and combined together in the end.

## Datasets

The table below summaries the dataset used during the project.

| Dataset        | Description    |  
| --------   | -----:   | 
| youtube_comments.ndjson.zst               | The dataset contains the features of more than 10B YouTube comments gathered from 24/05/2005 to 20/11/2019. | 
| df_channels_en.tsv.gz                     | The dataset contains the features of the 136’470 english YouTube channels.   |  
| video_to_channel_mapping_filtered.pkl     | The dataset contains for the channels having more than 300 comments a mapping between video_id and channel_id   |  



* youtube_comments: This dataset contains the following features for each video : `author_id` `id` `video_id` `crawled_at` `likes` `replies` `author` `content` 

* df_channels_en: This dataset contains the following features for each channel : `category` `join_date` `channel_id` `name` `subscribers_counts` `videos_counts` `subscriber_rank_sb` `weights`

* video_to_channel_mapping_filtered: `video_id` `channel_id`

### Files Description:


    ├── bipartite_graph                   # Folder corresponding to the bipartite graph implementation
    |   ├── ... not used for the final embedding
    |
    ├── helpers                           # Folder corresponding to the implementation of SGNS
    │   ├── create_helper_files.ipynb                                 # Create data if we want to modify the minimum number of comments per channel
    │   ├── config_threshold_value.py                                 # Parameters for the minimum number of comments per channel
    │   ├── helpers_channel_embedding.py                              # Helper file to compute the jumper and the position ratio
    │   ├── helpers_channels_more_10k.py                              # Helper file for channels having more than 10k comments
    │   ├── helpers_channels_more_300.py                              # Helper file for channels having more than 300 comments
    │   ├── helpers_channels_more_300.py                              # Helper file to visualize the axis projection method
    |
    ├── proximity_graph                   # Folder corresponding to the proximity graph implementation
    |   ├── ... not used for the final embedding
    |
    ├── word2vec                          # Folder using the word2vecf open source code
    |   ├── ... not used for the final embedding
    |
    ├── word2vec_pytorch                  # Folder corresponding to the implementation of SGNS (skip gram with negative sampling)
    │   ├── creates_training_data             # Folder where the training samples are constructed
    |   |   ├── preprocessing_file_gen_then_sample.ipynb              # Generates the pairs then sample
    |   |   ├── preprocessing_file_gen_then_weighted_sample.ipynb     # Generates the pairs then sample according to a weighted scheme
    |   |   ├── preprocessing_file_sample_then_gen.ipynb              # Sample channels then generate the pairs from the selected channels
    │   ├── embedding_space              # Folder keeping track of the results
    |   |   ├── ...
    │   ├── main.ipynb                                                # Notebook running the training process
    │   ├── model.py                                                  # Wor2Vec traditionnal model using pytorch
    │   ├── config.py                                                 # Parameters on the training and the result paths
    │   ├── utils_modified.py                                         # Helper file
    └── data_analysis.ipynb                                           # Notebook analysing the number of comments per channel            

Two configurations currently exist:
- Channels having more than 10k comments per channel
- Channels having more than 300 comments per channel

### How to run and create an embedding:

- Setup the parameters of the model under `word2vec_pytorch/config.py`.
- Create the training data under the folder `creates_training_data` by running one
of `preprocessing_file_gen_all_comb_then_sample.ipynb`,
`preprocessing_file_gen_all_comb_then_weighted_sample.ipynb`,
`preprocessing_file_sample_then_gen_comb.ipynb` depending on how you want
to generate the training samples.
- Run main.ipynb to create the channel embedding, the embedding will be stored in EMBEDDING_PATH (parameter in config.py)

### How to evaluate the performance of the embedding:
- If they do not exist, create `check_embedding_space.ipynb` and `axis_projection.ipynb` similar to the existing embeddings 
- Run `check_embedding_space.ipynb` and `axis_projection.ipynb` with the PATH being the path where the embedding is stored and with the correct `helpers/config_threshold_value.py` configuration.


### If you want to select channels having more than k comments per channel (k > 300):
- Modify the THRESHOLD and THRESHOLD_NAME parameter in `helpers/config_threshold_value.py` with the new threshold
- Run `data_analysis.ipynb`
- Run `helpers/create_helper_files.ipynb`
- Create a file similar to `helpers/helpers_channels_more_300.py` for the new THRESHOLD value.
- Run the part "How to run and create an embedding", importing the helper file created above
- Run the part "How to evaluate the performance of the embedding", importing the helper file created above
