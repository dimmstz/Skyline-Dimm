# Perfis de hardware e estratégia de desempenho — Skyline-Dimm

## Objetivo

Manter uma base Android comum e oferecer presets para aparelhos de entrada, intermediários e topo de linha. O app apresenta os presets nas configurações globais e sugere inicialmente um perfil aproximado usando RAM física total e quantidade de núcleos de CPU. A sugestão é apenas pré-selecionada: o app não aplica nem sobrescreve configurações existentes até o usuário escolher um perfil. A heurística não identifica o modelo da GPU nem substitui benchmark.

## Princípios globais

- Preservar NCE em ARM64 quando suportado; não trocar por interpreter/JIT genérico como "otimização".
- Manter Vulkan como primeira opção quando o driver expõe os recursos necessários; oferecer fallback por jogo quando houver incompatibilidade.
- Preservar cache persistente de shaders/pipelines. Não limpar caches a cada inicialização.
- Manter Async GPU e async presentation nos defaults upstream Android, evitando criar camadas extras de sincronização.
- Não aumentar indiscriminadamente workers, draw batches, precisão relaxada, clocks ou overclock. Mais concorrência pode piorar temperatura, 1% low e estabilidade.
- Texture unswizzling na GPU passa a ser habilitado por padrão em Android nesta proposta, mas continua configurável. Em GPU-bound devices, comparar desligado/ligado e manter desligado se o frametime piorar.
- Resolução abaixo de 1x é o controle mais direto para reduzir carga de pixel na GPU, mas não corrige gargalos de CPU, NCE, driver ou compilação de shaders.

## Perfil 1 — entrada / economia

Preset aplicado: resolução 0,5x, 2 pipeline workers, anisotropia padrão e GPU texture unswizzle desligado. Cache de shaders e compilação assíncrona permanecem habilitados. Indicado como ponto de partida para aparelhos com pouca RAM, CPU/GPU de entrada ou throttling rápido.

| Opção | Ponto de partida |
|---|---|
| Renderer | Vulkan se compatível; testar OpenGL por jogo |
| Resolução | 0,5x |
| GPU texture unswizzle | Desligado inicialmente; pode ser testado manualmente |
| Pipeline workers | 2 |
| Anisotropia | Desligada |
| Cache/compilação assíncrona | Ligados |
| Driver | Driver estável conhecido para aquela GPU |
| Precisão / hacks inseguros | Defaults; não forçar |
| Limite de FPS | 30 quando o jogo permitir e 60 não for sustentável |

Prioridade: reduzir stutter e manter frametime estável, não maximizar FPS médio a qualquer custo.

## Perfil 2 — intermediário / balanceado

Preset aplicado: resolução 0,75x, 2 pipeline workers, anisotropia 2x e GPU texture unswizzle ligado. É o perfil inicial escolhido para dispositivos com pelo menos 6 GB de RAM e 6 núcleos de CPU.

| Opção | Ponto de partida |
|---|---|
| Renderer | Vulkan |
| Resolução | 0,75x |
| GPU texture unswizzle | Ligado |
| Pipeline workers | 2 |
| Anisotropia | 2x |
| Async shaders | A/B por jogo |
| Driver | Sistema ou driver alternativo comprovadamente compatível |
| Precisão / hacks inseguros | Defaults |
| Limite de FPS | 30/60 conforme estabilidade do título |

## Perfil 3 — alto desempenho

Preset aplicado: resolução 1x, 4 pipeline workers, anisotropia 4x e GPU texture unswizzle ligado. A seleção inicial usa este perfil apenas com pelo menos 10 GB de RAM e 8 núcleos de CPU; aparelhos com GPU fraca ainda podem preferir o perfil intermediário.

| Opção | Ponto de partida |
|---|---|
| Renderer | Vulkan |
| Resolução | 1x |
| GPU texture unswizzle | Ligado |
| Pipeline workers | 4 |
| Anisotropia | 4x |
| Async shaders | A/B por jogo |
| Driver | Versão estável mais compatível; não presumir que a mais nova seja melhor |
| Precisão / hacks inseguros | Defaults |
| Limite de FPS | Nativo do jogo, se sustentado termicamente |

## Perfil de referência — Galaxy M23

- Samsung Galaxy M23 5G / SM-M236B
- Snapdragon 750G / Adreno 619
- ARM64, 8 GB RAM, Android 14 (referência de teste)

Pelo heurístico atual, o M23 (8 GB e 8 núcleos) inicia no perfil intermediário: resolução 0,75x, 2 pipeline workers, anisotropia 2x e GPU unswizzle ligado. Se a Adreno 619 ficar limitada pela GPU ou aquecer demais, selecionar Entrada/Economia ou desligar o unswizzle. Usar Vulkan e NCE quando compatíveis, driver estável por jogo, cache persistente e sem overclock.

## Alterações globais aplicadas

1. A tela principal de configurações oferece três presets globais selecionáveis.
2. Na primeira abertura das configurações, uma heurística simples de RAM física e núcleos pré-seleciona uma recomendação; nada é aplicado automaticamente. A escolha manual é salva e pode ser alterada.
3. Os presets alteram apenas resolução, pipeline workers, anisotropia, GPU unswizzle e opções de cache/compilação assíncrona.
4. Não são forçados hacks de precisão, overclock, clock máximo ou alterações de sincronização de memória.

## Como medir sem enganar

Para cada jogo e perfil:
1. Usar a mesma cena/save/câmera e a mesma versão do driver.
2. Fazer uma execução de aquecimento e três medições repetidas.
3. Registrar FPS médio, 1% low, frametime, temperatura, RAM e crashes.
4. Medir cache frio e cache quente separadamente.
5. Repetir por 15–20 minutos para detectar thermal throttling.
6. Aceitar uma mudança somente se melhorar frametime/1% low sem regressões gráficas ou de estabilidade.

Referências de frametime: 16,7 ms = 60 FPS; 33,3 ms = 30 FPS; 50 ms = 20 FPS.

## Limitações conhecidas

- A recomendação inicial considera apenas RAM física e número de núcleos; ela não detecta arquitetura, GPU, largura de banda, driver ou capacidade térmica. É um ponto de partida, não um classificador definitivo. Os presets só alteram configurações depois da confirmação do usuário.
- O ganho de FPS do unswizzle não está medido no M23 nem em uma matriz ampla de aparelhos.
- O build CI confirma compilação e validação do APK, não desempenho real. É necessário benchmark em aparelhos fracos, médios e topo de linha.
- O desempenho sustentado depende de driver, jogo, temperatura, resolução, shaders e estado do sistema.
