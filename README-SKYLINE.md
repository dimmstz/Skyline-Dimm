# Skyline-Dimm — Android performance builds

Fork de testes do Eden com foco em desempenho, compatibilidade e estabilidade Android.

- Upstream: [eden-emu/eden](https://git.eden-emu.dev/eden-emu/eden), branch `master`, com submódulos recursivos.
- Workflow: [Android M23 Release](.github/workflows/android-m23-release.yml).
- Perfis de hardware no app: [entrada, intermediário e alto desempenho](ANDROID-HARDWARE-PERFORMANCE-PROFILES.md).
- Base de análise anterior: [M23 performance profile](M23-PERFORMANCE-PROFILE.md).

## Sobre desempenho

Os perfis ficam nas configurações globais. Uma heurística baseada em RAM física e núcleos pré-seleciona uma recomendação, mas não altera as configurações até o usuário aplicar o perfil. O CI compila e valida APK + AAB ARM64, incluindo assinatura/alinhamento do APK e checksums SHA-256; não mede FPS real. Nenhum ganho de desempenho é declarado sem teste A/B com FPS médio, 1% low, frametime e temperatura.

## Distribuição

Builds concluídos na branch `main` publicam automaticamente a tag `m23-build-N`, a release GitHub com APK, AAB e `SHA256SUMS.txt`. Builds de pull request geram artefatos de validação, sem publicar uma release prematuramente.

## Testes de aceitação

1. Instalar APK ARM64 em aparelhos de entrada, intermediários e topo de linha.
2. Testar Vulkan e fallback gráfico por jogo.
3. Comparar GPU texture unswizzling ligado/desligado.
4. Registrar FPS, 1% low, frametime, RAM e temperatura em sessões de 15–20 minutos.
5. Rejeitar defaults que provoquem crashes, artefatos, thermal throttling ou piora de frametime.
