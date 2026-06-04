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



BLIP implementado, converte as imagens em captions

Resumo: `top_k` e `top_p` (nucleus sampling)

- `top_k`: limita o conjunto de tokens candidatos aos K tokens mais prováveis antes de amostrar. Cortes mais pequenos (K baixo) reduzem diversidade e tornam a geração mais conservadora; K alto aumenta variedade.
- `top_p` (nucleus sampling): ordena tokens por probabilidade e escolhe o menor subconjunto cujo somatório de probabilidades ≥ p; a amostragem é feita apenas desse "núcleo". `top_p` adapta dinamicamente o tamanho do conjunto de escolha, equilibrando qualidade e diversidade.
- Interação: usar ambos fornece um corte duro (`top_k`) e uma seleção dinâmica (`top_p`). A temperatura (`temperature`) escala probabilidades antes da amostragem e controla o nível de aleatoriedade.

Recomendação prática para BLIP no TP2: `do_sample=True, top_k=50, top_p=0.95, temperature=0.9, num_return_sequences=2-3` — ajusta para menos se houver limitações de VRAM/tempo.