# Perfil de desempenho — Galaxy M23

## Hardware de referência

- Samsung Galaxy M23 5G (SM-M236B)
- Qualcomm Snapdragon 750G
- GPU Qualcomm Adreno 619
- Android 14 / API 34
- Memória física: 8 GB
- ABI: arm64-v8a

Este documento é um perfil inicial conservador, não uma alegação de que o emulador já foi otimizado ou validado em todos os jogos nesse aparelho.

## Configuração inicial recomendada

| Opção | Ponto de partida | Observação |
|---|---|---|
| Backend gráfico | Vulkan | Testar OpenGL por jogo se houver artefatos, falhas ou fechamento |
| Driver | Driver Qualcomm de sistema; Turnip somente em versão testada | Não presumir que a versão mais nova seja a mais rápida ou estável |
| Resolução | 1x | Reduzir para 0,75x ou 0,5x somente se a build oferecer e houver gargalo de GPU |
| Async GPU | Ligado como primeiro teste | Comparar ligado/desligado no mesmo trecho; pode causar incompatibilidades específicas |
| NVDEC | CPU como primeiro teste no Android | Priorizar estabilidade de vídeos/cutscenes; comparar se houver opção alternativa |
| Áudio | SDL3, se disponível | Confirmar ausência de estalos e dessincronização |
| Precisão de CPU/GPU | Padrão/recomendado | Não desativar verificações de precisão globalmente |
| Workers de pipelines Vulkan | 2 como padrão; testar 3 e 4 | Mais workers aceleram compilação, mas aumentam calor; não usar 8 sem medição |\n| Shaders assíncronos | Testar ligado/desligado por jogo | Pode reduzir travadas de compilação, mas causar pop-in; não garante FPS sustentado |\n| Compilação/cache de shaders | Cache persistente habilitado, se disponível | Evitar limpar cache repetidamente; primeiro carregamento pode apresentar stutter |
| Overclock/forçar clocks | Desativado | Evitar calor, consumo e throttling |

## Compatibilidade e limites

- Vulkan 1.3 anunciado pelo driver não significa suporte a todas as extensões ou limites exigidos pelo emulador.
- Mensagens como `wideLines` ausente ou limites de bindings insuficientes indicam recursos não expostos pelo driver. Não devem ser contornadas forçando recursos sem validação.
- Turnip pode funcionar no Adreno 619, mas a compatibilidade depende da versão do driver, Android e build. Um driver que inicializa não prova estabilidade em jogos.
- Opções experimentais como Fastmem/Host MMU, FMA/ASIMD inseguro, desativação de verificações de endereço ou alterações de sincronização devem permanecer nos padrões até teste individual. Podem produzir erros gráficos, corrupção ou encerramentos.
- A resolução abaixo de 1x reduz carga de GPU, mas não corrige gargalos de CPU, emulação ou compilação de shaders.

## Protocolo A/B obrigatório

1. Escolher um jogo legalmente obtido e uma cena reproduzível de 3 a 5 minutos.
2. Reiniciar o jogo entre testes e manter driver, resolução, energia e temperatura inicial tão semelhantes quanto possível.
3. Alterar somente uma opção por vez.
4. Registrar FPS médio, 1% low, frametime, temperatura, consumo de memória e ocorrência de falhas.
5. Repetir cada cenário pelo menos três vezes.
6. Rejeitar a alteração se melhorar FPS médio mas piorar muito o 1% low, causar artefatos, crash ou throttling.
7. Repetir após 15–20 minutos para detectar perda de desempenho por aquecimento.

## O que ainda não está comprovado

Este repositório não contém, até o momento, benchmark controlado que demonstre ganhos próprios de CPU/GPU no M23. A seleção de configurações acima é ponto de partida para ensaios, não promessa de FPS ou compatibilidade universal.
