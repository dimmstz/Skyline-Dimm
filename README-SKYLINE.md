# Skyline-Dimm — Android performance builds

Fork de testes do Eden com foco em desempenho, compatibilidade e estabilidade Android.

- Upstream: [eden-emu/eden](https://git.eden-emu.dev/eden-emu/eden), branch `master`, com submódulos recursivos.
- Workflow: [Android M23 Release](.github/workflows/android-m23-release.yml).
- Perfil de hardware: [fracos, médios e topo de linha](ANDROID-HARDWARE-PERFORMANCE-PROFILES.md).
- Base de análise anterior: [M23 performance profile](M23-PERFORMANCE-PROFILE.md).

## Sobre desempenho

Os perfis são pontos de partida e devem ser medidos por jogo. O CI confirma compilação e integridade do APK, mas não mede FPS real. Nenhum ganho de desempenho é declarado sem teste A/B com FPS médio, 1% low, frametime e temperatura.

## Testes de aceitação

1. Instalar APK ARM64 em aparelhos de entrada, intermediários e topo de linha.
2. Testar Vulkan e fallback gráfico por jogo.
3. Comparar GPU texture unswizzling ligado/desligado.
4. Registrar FPS, 1% low, frametime, RAM e temperatura em sessões de 15–20 minutos.
5. Rejeitar defaults que provoquem crashes, artefatos, thermal throttling ou piora de frametime.
