# Análise técnica de desempenho — Skyline Dimm / Galaxy M23

**Data da análise:** 2026-11-10  
**Base analisada:** código-fonte espelhado do Eden em `eden-emulator/mirror`, além do workflow e perfil deste repositório.  
**Dispositivo-alvo:** Galaxy M23 5G / SM-M236B / Snapdragon 750G / Adreno 619 / 8 GB / Android 14 / ARM64.

> Esta análise identifica caminhos reais no código e propostas de ensaio. Não representa benchmark executado no M23, nem afirma ganhos de FPS ainda não medidos.

## 1. Limite de hardware e expectativa

A tabela oficial de requisitos do Eden indica Snapdragon 865 / Adreno 650 como mínimo para Android. O Snapdragon 750G / Adreno 619 do M23 fica abaixo desse patamar. Isso não impede a execução de homebrew ou jogos leves, mas limita a expectativa para títulos 3D pesados. A meta correta é estabilidade, frametime consistente e o melhor desempenho possível por jogo, não prometer 60 FPS universais.

Referência: https://eden-emu.dev/system-requirements/

A documentação de arquitetura do Eden afirma que, em ARM64 com NCE, a GPU tende a ser o gargalo mais frequente no Android; também recomenda NCE quando disponível. Para o M23, isso significa que reduzir resolução pode ajudar em cenários limitados pela GPU, mas não resolve gargalo de CPU, emulação, shaders ou sincronização.

Referência: https://github.com/eden-emulator/mirror/blob/master/docs/user/Architectures.md

## 2. Achados confirmados no código

### 2.1 NCE / CPU

Arquivo: `src/core/arm/nce/arm_nce.cpp`

O projeto contém implementação de Native Code Execution para ARM64/Android. A recomendação é preservar NCE como caminho principal quando suportado. Não trocar por JIT/interpreter como suposta otimização. Não alterar rotinas de sinais, contexto ou sincronização do NCE sem benchmark e testes de estabilidade: erros nessa camada podem encerrar o processo ou corromper a execução.

A configuração `cpuopt_unsafe_host_mmu` tem valor padrão habilitado nas plataformas Android/Linux/afins no arquivo `src/common/settings.h`. Não é recomendável forçar opções inseguras adicionais globalmente. Qualquer experimento deve ser isolado por jogo e reversível.

### 2.2 Número de workers de pipelines Vulkan

Arquivos:
- `src/android/app/src/main/jni/android_settings.h`
- `src/video_core/renderer_vulkan/vk_pipeline_cache.cpp`

O valor padrão de `pipeline_worker_count` no Android é **2**. O código limita a configuração entre 2 e 8 e também considera `hardware_concurrency - 1`. Se o driver for identificado com compilação paralela problemática, o construtor do cache usa somente 1 worker.

A thread pool de pipelines é criada com posicionamento de background. Mais workers podem reduzir o tempo de compilação de pipelines/shaders, mas aumentam concorrência e calor. Não há evidência de que aumentar esse número aumente o FPS sustentado depois que os pipelines já estão compilados.

**Plano A/B recomendado:** comparar 2, 3 e 4 workers em uma cena que compile muitos shaders; registrar tempo até estabilizar, stutter, temperatura e FPS/frametime após aquecer o cache. Não iniciar com 8. Manter 2 como padrão até os resultados do M23 justificarem outro valor.

### 2.3 Compilação assíncrona de shaders

Arquivos:
- `src/common/settings.h`
- `src/video_core/renderer_vulkan/vk_pipeline_cache.cpp`

`use_asynchronous_shaders` tem padrão falso em `settings.h`. Quando habilitada, o método `PipelineCache::BuiltPipeline` pode adiar certos pipelines não compilados para evitar bloquear o fluxo de renderização. Há exceções deliberadas para render targets pequenos, efeitos colaterais, transform feedback, rasterização desativada e draws pequenos.

**O que pode melhorar:** picos de frametime durante a primeira visita a áreas/efeitos e a sensação de travamento durante compilação.

**O que não promete:** aumento permanente de FPS em uma cena já aquecida. A renderização pode apresentar objetos temporariamente ausentes ou pop-in, e alguns jogos podem ser sensíveis à ordem de execução.

**Plano A/B:** comparar desligado/ligado por jogo, sempre com cache frio e depois cache quente. Registrar 1% low e picos de frametime, não só FPS médio. Não mudar o padrão global sem evidência.

### 2.4 Emulação GPU assíncrona e apresentação Vulkan

Arquivo: `src/common/settings.h`

`use_asynchronous_gpu_emulation` tem padrão verdadeiro. `async_presentation` também é definido como verdadeiro no Android. Portanto, ativar novamente essas opções não seria uma modificação inédita do Skyline. O changelog upstream também documenta defaults Android para Async GPU ligado, NVDEC CPU e SDL3.

Referência: https://git.eden-emu.dev/eden-emu/eden/pulls/4509

Não adicionar outra camada de threads ou alterar a sincronização de apresentação sem identificar um problema reproduzível. Mudanças na cadência de frames podem trocar FPS aparente por stutter, input lag ou dessincronização.

### 2.5 Lote de draws no rasterizador Vulkan

Arquivo: `src/video_core/renderer_vulkan/vk_rasterizer.cpp`

O caminho Android usa `DRAWS_TO_DISPATCH = 512`, enquanto o caminho não Android usa 4096. Isso mostra que o upstream já usa um lote bem menor no Android, provavelmente para equilibrar latência, memória e submissão de trabalho.

Não aumentar 512 para 4096 por tentativa. Um lote maior pode reduzir overhead de submissão em alguns cenários, mas também atrasar o despacho, aumentar picos de memória e piorar frametime em GPU móvel. Só testar com métricas de CPU/GPU e vários jogos.

### 2.6 Cache de pipelines e shaders

Arquivo: `src/video_core/renderer_vulkan/vk_pipeline_cache.cpp`

O código serializa o cache de pipelines Vulkan em background quando o cache do driver está habilitado. Preservar cache persistente é importante para reduzir recompilação entre sessões. Limpar cache repetidamente pode piorar stutter; limpar apenas após troca de driver ou quando houver corrupção/artefatos compatíveis com cache inválido.

## 3. Diagnóstico por overlay

O código Android contém opções para mostrar:
- FPS;
- frametime;
- RAM do processo e RAM do sistema;
- temperatura da bateria;
- consumo/potência;
- quantidade de shaders em compilação;
- modelo do aparelho, SoC, GPU e versão do driver.

Usar FPS junto de frametime é essencial. Referências aproximadas:
- 60 FPS: 16,7 ms por frame;
- 30 FPS: 33,3 ms por frame;
- 20 FPS: 50 ms por frame.

Se o FPS médio parece aceitável, mas o frametime tem picos frequentes, o jogo ainda pode parecer travado. Ativar somente os indicadores necessários durante a medição, pois overlays e telemetria também podem adicionar algum overhead.

## 4. Plano de testes recomendado para o M23

1. **Baseline:** Vulkan, driver de sistema Qualcomm, NCE, Async GPU no padrão, NVDEC CPU no Android, SDL3 se disponível, resolução 1x, cache persistente. Não alterar várias opções simultaneamente.
2. **Resolução:** comparar 1x, 0,75x e 0,5x somente se a build oferecer esses valores. Se FPS/frametime melhorarem muito, há componente de gargalo de GPU. Se quase nada mudar, investigar CPU/driver/shaders.
3. **Pipeline workers:** 2 vs 3 vs 4 em cena com compilação; depois comparar desempenho com cache quente. Priorizar temperatura e 1% low.
4. **Async shaders:** ligado/desligado por jogo; observar pop-in, crashes, stutter de primeira execução e estabilidade depois do cache aquecido.
5. **Driver:** comparar driver Qualcomm e uma versão Turnip compatível, sem presumir que Turnip será mais rápido. Ao trocar driver, esperar recompilação de shaders e pipelines.
6. **Sessão térmica:** repetir o teste por 15–20 minutos. Se o desempenho cair progressivamente, provável limitação térmica/energia; aumentar threads ou clocks tende a piorar o resultado sustentado.
7. **Repetibilidade:** três execuções por cenário, mesmo save/área/câmera, temperatura inicial semelhante e uma variável por vez.

## 5. Ordem de implementação — risco x benefício

| Prioridade | Ação | Benefício esperado | Risco |
|---|---|---|---|
| P0 | Expor/usar overlay FPS + frametime + temperatura + RAM | Identificar gargalo real | Baixo |
| P0 | Perfil por jogo para resolução e Async shaders | Ajustar sem prejudicar todos os jogos | Baixo/médio |
| P1 | A/B workers 2/3/4 | Menos tempo de compilação, possível redução de stutter inicial | Médio, calor |
| P1 | Teste de drivers Qualcomm vs Turnip por jogo | Melhorar compatibilidade ou estabilidade | Médio/alto |
| P2 | Instrumentação de tempos CPU, GPU e espera de fences | Encontrar gargalo no código com precisão | Baixo impacto em release; requer desenvolvimento |
| P2 | Ajustar lote Android 512 somente após perfil de GPU/CPU | Possível redução de overhead | Alto risco de regressão |
| P3 | Alterações em NCE, sincronização, allocator ou precisão | Potencial ganho específico | Muito alto |

## 6. O que não fazer

- Não forçar recursos Vulkan ausentes no driver.
- Não presumir que Vulkan 1.3 significa suporte a todas as extensões/limites exigidos.
- Não usar 8 workers ou clocks forçados como padrão no M23.
- Não reduzir resolução esperando corrigir gargalo de CPU.
- Não desativar precisão, validações, barreiras ou sincronização globalmente.
- Não limpar cache a cada execução.
- Não considerar build Release, sozinha, como prova de FPS maior; ela reduz overhead de debug, mas não corrige gargalos do renderer.

## 7. Conclusão

Os maiores ganhos realistas e seguros no curto prazo são: medir corretamente, preservar NCE, manter o caminho Vulkan estável, ajustar resolução por jogo, comparar Async shaders e testar 2/3/4 workers com controle térmico. O código upstream já ativa Async GPU e async presentation no Android e já reduz o lote de draws para 512; duplicar essas mudanças não traria ganho.

Ainda não existe benchmark do M23 que permita escolher um novo valor padrão de workers, alterar o lote de draws ou prometer ganhos de FPS. O próximo passo de engenharia deve ser adicionar instrumentação de frametime/espera CPU-GPU e executar A/B no próprio aparelho antes de aplicar patches de baixo nível.
