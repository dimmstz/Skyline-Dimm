# Perfis de hardware e estratégia de desempenho — Skyline-Dimm

## Objetivo

Manter uma base Android comum e oferecer pontos de partida distintos para aparelhos fracos, médios e topo de linha. Os perfis abaixo são recomendações iniciais, não detecção automática nem promessa de FPS. A seleção automática por SoC/GPU ainda precisa ser implementada e validada no app.

## Princípios globais

- Preservar NCE em ARM64 quando suportado; não trocar por interpreter/JIT genérico como "otimização".
- Manter Vulkan como primeira opção quando o driver expõe os recursos necessários; oferecer fallback por jogo quando houver incompatibilidade.
- Preservar cache persistente de shaders/pipelines. Não limpar caches a cada inicialização.
- Manter Async GPU e async presentation nos defaults upstream Android, evitando criar camadas extras de sincronização.
- Não aumentar indiscriminadamente workers, draw batches, precisão relaxada, clocks ou overclock. Mais concorrência pode piorar temperatura, 1% low e estabilidade.
- Texture unswizzling na GPU passa a ser habilitado por padrão em Android nesta proposta, mas continua configurável. Em GPU-bound devices, comparar desligado/ligado e manter desligado se o frametime piorar.
- Resolução abaixo de 1x é o controle mais direto para reduzir carga de pixel na GPU, mas não corrige gargalos de CPU, NCE, driver ou compilação de shaders.

## Perfil 1 — aparelhos fracos / entrada

Critérios indicativos: SoC antigo ou de entrada, GPU com baixa largura de banda, 4–6 GB RAM, throttling rápido.

| Opção | Ponto de partida |
|---|---|
| Renderer | Vulkan se compatível; testar OpenGL por jogo |
| Resolução | 0,5x–0,75x, se disponível |
| GPU texture unswizzle | A/B; desligar se GPU-bound |
| Pipeline workers | 2 |
| Async shaders | Testar por jogo; priorizar estabilidade |
| Driver | Driver estável conhecido para aquela GPU |
| Precisão / hacks inseguros | Defaults; não forçar |
| Limite de FPS | 30 quando o jogo permitir e 60 não for sustentável |

Prioridade: reduzir stutter e manter frametime estável, não maximizar FPS médio a qualquer custo.

## Perfil 2 — aparelhos médios

Critérios indicativos: SoC intermediário recente, GPU intermediária, 6–8 GB RAM.

| Opção | Ponto de partida |
|---|---|
| Renderer | Vulkan |
| Resolução | 0,75x–1x |
| GPU texture unswizzle | Ligado inicialmente; comparar A/B |
| Pipeline workers | 2; testar 3 em cenas com compilação |
| Async shaders | A/B por jogo |
| Driver | Sistema ou driver alternativo comprovadamente compatível |
| Precisão / hacks inseguros | Defaults |
| Limite de FPS | 30/60 conforme estabilidade do título |

## Perfil 3 — topo de linha

Critérios indicativos: SoC flagship recente, GPU alta, 8–16 GB RAM e boa sustentação térmica.

| Opção | Ponto de partida |
|---|---|
| Renderer | Vulkan |
| Resolução | 1x; testar acima somente se houver opção |
| GPU texture unswizzle | Ligado; comparar em jogos GPU-bound |
| Pipeline workers | 2; testar 3–4 com benchmark |
| Async shaders | A/B por jogo |
| Driver | Versão estável mais compatível; não presumir que a mais nova seja melhor |
| Precisão / hacks inseguros | Defaults |
| Limite de FPS | Nativo do jogo, se sustentado termicamente |

## Perfil de referência — Galaxy M23

- Samsung Galaxy M23 5G / SM-M236B
- Snapdragon 750G / Adreno 619
- ARM64, 8 GB RAM, Android 14 (referência de teste)

Ponto de partida conservador: Vulkan quando compatível, driver Qualcomm ou Turnip comprovadamente estável para o jogo, NCE, resolução 0,5x–0,75x se houver gargalo de GPU, 2 pipeline workers, cache persistente e sem overclock. A opção de GPU unswizzle deve ser comparada ligada/desligada porque pode deslocar carga da CPU para a Adreno 619.

## Alterações globais que esta proposta introduz

1. Android passa a iniciar com GPU texture unswizzling habilitado, mantendo o setting reversível pelo usuário.
2. Perfis de hardware e critérios de escolha ficam documentados para orientar presets e testes.
3. Não são alterados os caminhos de sincronização, NCE, precisão ou limites Vulkan sem evidência de benchmark.

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

- Os perfis são documentação de tuning; esta PR não adiciona ainda uma tela de seleção automática no Android.
- O ganho de FPS do unswizzle não está medido no M23 nem em uma matriz ampla de aparelhos.
- O build CI confirma compilação e validação do APK, não desempenho real. É necessário benchmark em aparelhos fracos, médios e topo de linha.
- O desempenho sustentado depende de driver, jogo, temperatura, resolução, shaders e estado do sistema.
