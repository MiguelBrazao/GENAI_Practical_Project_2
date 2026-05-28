Usados estes comandos para criar ambiente conda:

conda create -n GEN_AI_TP2 python=3.11 -y
conda activate GEN_AI_TP2
conda install pytorch pytorch-cuda=12.1 -c pytorch -c nvidia -y
python -m pip install "diffusers==0.35.2" "transformers<5"

deu problema com versão de sympy, tinha 1.14.o mas tinha que ser 1.13.1
fiz python -m pip install "sympy==1.13.1"

python -m pip install "diffusers==0.35.2" "transformers<5"
python -m pip install accelerate safetensors matplotlib torchvision==0.20.1 ipywidgets "pandas<3" "Pillow<12" numpy

Problemas que apareceram no notebook:
- `FrozenInstanceError` ao tentar alterar `config.num_inference_steps` na célula do exemplo mínimo. O problema estava em modificar diretamente o `config`, que é imutável. Para resolver, foi usado `from dataclasses import replace` e criado `test_config = replace(config, ...)`.
- Imagem toda preta com `generated image range: min=0.0000, max=0.0000`. O problema estava em o kernel ainda usar uma versão antiga da função `render_prompt`; para resolver, foi recarregada a célula certa da função e o exemplo foi executado novamente.
- Versão antiga de `render_prompt`: a saída do pipeline estava a ser tratada de forma pouco segura e sem diagnóstico suficiente. Na versão nova, passei a usar `output_type="np"`, adicionei normalização defensiva com `nan_to_num`, `clip` e reescala quando necessário, e imprimi `dtype`, `shape`, `min`, `max`, `mean` e `nonzero` para confirmar a saída antes de guardar a imagem; depois de recarregar a célula da função, a imagem deixou de sair preta.
