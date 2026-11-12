# Skyline Dimm — Galaxy M23

Build experimental do Eden para Android, com foco de compatibilidade e testes no Samsung Galaxy M23 5G.

## Dispositivo-alvo

- Modelo: Samsung Galaxy M23 5G (SM-M236B)
- SoC: Qualcomm Snapdragon 750G
- GPU: Adreno 619
- Android de referência: Android 14
- ABI: arm64-v8a

## Estado do projeto

Este repositório mantém o perfil do dispositivo e a automação de compilação. O código do emulador é obtido do repositório upstream do Eden durante o workflow; portanto, ainda não representa um fork com otimizações próprias de CPU/GPU já comprovadas no Galaxy M23.

As alterações específicas do aparelho devem ser pequenas, isoladas e acompanhadas de testes A/B. Não considerar uma configuração mais rápida apenas pelo nome ou por relatos de outros dispositivos.

## Perfil de desempenho

Consulte [M23-PERFORMANCE-PROFILE.md](./M23-PERFORMANCE-PROFILE.md) para as configurações iniciais e [M23-DEEP-PERFORMANCE-ANALYSIS.md](./M23-DEEP-PERFORMANCE-ANALYSIS.md) para a análise do código upstream, gargalos e plano A/B.

## Compilação Android

O workflow **Android M23 Release** pode ser executado manualmente em Actions e também é acionado por pushes na branch `main`. O push feito pelo próprio bot durante a sincronização do upstream não inicia outra compilação, evitando loop de execuções.

A compilação atual usa o alvo Android `legacy` do script upstream e gera APK ARM64. Isso é uma escolha de build, não uma garantia de maior desempenho. O workflow valida metadados, alinhamento, assinatura e presença de bibliotecas ARM64; ele não mede FPS nem estabilidade em aparelho físico.

Os artefatos são publicados na execução do GitHub Actions por 14 dias.

## Testes necessários no aparelho

- FPS médio e 1% low em uma cena reproduzível.
- Frametime e travamentos após aquecimento de shaders.
- Temperatura e possível thermal throttling após 15–20 minutos.
- Comparação Vulkan/Turnip e OpenGL por jogo.
- RAM e encerramentos inesperados.
- Versão do APK, driver gráfico, jogo e configurações usadas.

## Licença

O código upstream do Eden mantém seus avisos e termos de licença. Consulte os arquivos de licença do upstream antes de redistribuir builds.
