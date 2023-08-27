PEFT_SHARE_BASE_WEIGHTS=true python3 -m fastchat.serve.multi_model_worker \
    --model-path /home/work/.cache/huggingface/modules/transformers_modules/THUDM/chatglm-6b-int4 \
    --model-names chatglm-6b-int4 \
    --model-path /data/chris/peft-llama-dummy-2 \
    --model-names peft-dummy-2 \
    --model-path /data/chris/peft-llama-dummy-3 \
    --model-names peft-dummy-3 \
    --num-gpus