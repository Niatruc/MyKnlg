# 环境搭建
* Windows
    * cuda
        * `nvidia-smi`查看最高支持的cuda版本
        * 下载: https://developer.nvidia.com/cuda-toolkit-archive
    * cudnn
        * 下载: https://developer.nvidia.com/cudnn-archive
        * 若下了tar包, 将其中`bin`, `include`, `lib`目录下的文件分别拷贝到CUDA目录下的各对应目录. 
# 库, 框架
* huggingface
    * `pip install huggingface_hub`
    * token: 命令行运行`huggingface-cli.exe login`, 按提示输入在huggingface网站上创建的token. 

    ```py
        from huggingface_hub import snapshot_download

        # 下载gemma-2b模型
        snapshot_download(
            repo_id="google/gemma-2b-flax", local_dir=r"D:\models\huggingface\hub\models--google--gemma-2b-flax\snapshots\4b7a4a7ffe3b60b6e6e995f0c925b84ce128072f"
        )

        # 下载的文件: 
            # 2b/              # Directory containing model weights
            # tokenizer.model  # Tokenizer
    ```

* ollama
    * 参考
        * [ollama-python](https://github.com/ollama/ollama-python)
    * 模型存放位置环境变量: `OLLAMA_MODELS`
    * 命令
        * `list`: 列出已有模型
        * `serve`: 开启服务
        * `run <模型>`

* tensorflow
    * 注
        * Tensorflow 2.11(含)以后Windows版本不支持GPU
    * 安装
        * windows
            * 参考: [在Win10/11 建立Tensorflow GPU訓練環境20230605](https://hackmd.io/@jerrychu/S1QvFG98h)
            * 环境: 
                * windows 11
                * python 3.9
                * cuda 11.8
                * cudnn 8.9.7
                * tensorflow 2.10: `pip install tensorflow==2.10`
            * 问题
                * `A module that was compiled using NumPy 1.x cannot be run in NumPy 2.0.1 as it may crash.`
                    * numpy 2 的问题. 改装numpy 1: `pip install "numpy<2.0"`
    * 使用
        ```py
            from tensorflow.python.client import device_lib
            print(device_lib.list_local_devices()) # 测试gpu的可用性

            import tensorflow as tf

            print("TensorFlow version:", tf.__version__) # tf的版本
            print("Is TensorFlow built with CUDA:", tf.test.is_built_with_cuda()) # 是否可用cuda
            print("Is GPU available:", tf.test.is_gpu_available()) # 是否可用gpu
        ```
* pytorch
    * 安装
        * https://pytorch.org/get-started/locally/
        * 从上述链接中按提示选好环境, 复制所给pip命令, 安装pytorch
            * `cuda 12.1`对应的pytorch在安装时会带有`cudart64_12.dll`, `cudnn_cnn_train64_8.dll`等动态库(在`Lib\site-packages\torch\lib`下)
    
    * 使用
        ```py
        
            import torch.utils.cpp_extension
            torch.utils.cpp_extension.CUDA_HOME # Pytorch 实际使用的运行时的 cuda 目录
            
            import torch
            torch.version.cuda # 编译该 Pytorch release 版本时使用的 cuda 版本

            torch.cuda.is_available() # 判断cuda是否可用
            torch.cuda.device_count() # 可用cuda的设备数量
        ```

## gemma
```py
```

* `python examples/sampling.py --path_checkpoint=<模型路径>/2b/ --path_tokenizer=<模型路径>/tokenizer.model`