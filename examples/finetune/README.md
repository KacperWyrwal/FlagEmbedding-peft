# Finetune
In this example, we show how to finetune the baai-general-embedding with your data.

## Installation
* **with pip**
```
pip install -U FlagEmbedding
```

* **from source**
```
git clone https://github.com/FlagOpen/FlagEmbedding.git
cd FlagEmbedding
pip install  .
```
For development, install as editable:
```
pip install -e .
```
 

## Data format
Train data should be a json file, where each line is a dict like this:

```
{"query": str, "pos": List[str], "neg":List[str]}
```

`query` is the query, and `pos` is a list of positive texts, `neg` is a list of negative texts.
If you have no negative texts for a query, you can random sample some from the entire corpus as the negatives.

See [toy_finetune_data.jsonl]() for a toy data file.


## Train
```
torchrun --nproc_per_node {number of gpus} \
-m FlagEmbedding.baai_general_embedding.finetune.run \
--output_dir {path to save model} \
--model_name_or_path BAAI/bge-large-en \
--train_data toy_finetune_data.jsonl \
--learning_rate 1e-5 \
--num_train_epochs 5 \
--per_device_train_batch_size 1 \
--dataloader_drop_last True \
--normlized True \
--temperature 0.01 \
--query_max_len 32 \
--passage_max_len 128 \
--negatives_cross_device 
```

**some important arguments**:
- `train_group_size`: the number of positive and negatives for a query in training.
There are always one postive, so this argument will control the number of negatives (#negatives=train_group_size-1).
Noted that the number of negatives should not be larger than the numbers of negatives in data `"neg":List[str]`.
Besides the negatives in group, the in-batch negatives also will be used in fine-tuning.
- `negatives_cross_device`: share the negatives across all GPUs. This argument will extend the number of negatives.
- `per_device_train_batch_size`: batch size in training. In most of cases, larger batch size will bring stronger performance.
- `learning_rate`: select a appropriate for your model. Recommend 1e-5/2e-5/3e-5 for large/base/small-scale. 

More training arguments please refer to [transformers.TrainingArguments](https://huggingface.co/docs/transformers/main_classes/trainer#transformers.TrainingArguments)





