# Skyline Dimm — perfil Galaxy M23

Projeto experimental baseado no **Eden** (Nintendo Switch), com pipeline próprio para gerar APK Android e documentação de validação no Samsung Galaxy M23.

## Origem e licença

- Upstream: https://git.eden-emu.dev/eden-emu/eden
- Branch acompanhada: `master`
- Licença upstream: GNU GPL-3.0-or-later. Este projeto deve preservar avisos de copyright, licença e atribuição do Eden e de seus contribuidores.
- O código completo do Eden é obtido do upstream durante o workflow; este repositório mantém a automação e o perfil de dispositivo. Isso evita copiar uma árvore upstream grande e desatualizada para este repositório.

## Dispositivo-alvo

Galaxy M23 5G (SM-M236B), Snapdragon 750G / Adreno 619, Android 14, ARM64.

### Perfil inicial conservador

Estas são recomendações iniciais de teste, não alterações mágicas no motor nem garantia de FPS:

1. Usar a renderização Vulkan quando o jogo e o driver Qualcomm do aparelho forem estáveis; comparar com OpenGL por jogo se houver falhas.
2. Começar em resolução 1x (nativa) e testar 0,75x/0,5x somente se a build oferecer essas opções e o jogo estiver limitado pela GPU.
3. Manter precisão/compilação de shaders nos padrões recomendados pelo emulador; não desativar verificações de precisão globalmente.
4. Fechar apps em segundo plano, manter espaço livre e evitar jogar enquanto o aparelho estiver quente/carregando.
5. Registrar FPS, frametime, temperatura, jogo, versão do driver e backend. Comparar sempre a mesma cena e a mesma configuração.
6. Não forçar drivers Vulkan de terceiros sem confirmar compatibilidade com Adreno 619 e com a versão do Android.

## Build Android

O workflow **Android M23 Release** baixa o Eden upstream com submódulos e executa o script oficial de build Android no flavor `standard`, tipo `Release`. Os APKs/AABs gerados são publicados como artefatos do GitHub Actions.

- Execução manual: aba Actions → Android M23 Release → Run workflow.
- O workflow registra o commit exato do upstream usado.
- Build falha ou artefato ausente deve ser tratado como falha; não presumir sucesso.

## Limitações desta primeira etapa

O repositório ainda não contém um fork integral nem patches internos de CPU/GPU do Eden. Não é seguro inventar otimizações C++ sem perfis de execução e testes A/B no aparelho. A primeira etapa estabelece build reproduzível e documentação do perfil; otimizações no motor devem entrar em patches pequenos, medidos e validados no M23.

## Próximas medições necessárias

- FPS médio e 1% low em cenas reproduzíveis.
- Frametime e stutter durante compilação de shaders.
- Temperatura/thermal throttling após 15–20 minutos.
- Comparativo Vulkan × OpenGL por jogo.
- Consumo de RAM e encerramentos por pressão de memória.

