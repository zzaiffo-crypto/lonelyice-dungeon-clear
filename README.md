# Dungeon Clear para LonelyIce — tentativa automatizada de compilação

**Estado: experimental.** Não é uma DLL pronta. Nunca foi compilado/testado nesta sessão. O workflow usa Windows 2022, MSVC, LonelyIce v0.1.1 e Playerbots da mesma árvore de código. A compatibilidade com a sua instalação ainda depende de teste.

## Como usar sem instalar compiladores no notebook

1. No GitHub, crie um repositório privado vazio, por exemplo `lonelyice-dungeon-clear`.
2. Adicione os arquivos deste ZIP preservando os caminhos (inclusive `.github/workflows/compilar.yml`). Se seu navegador não permitir enviar pastas, instale o **GitHub Desktop** e publique a pasta inteira, ou crie os arquivos nos caminhos indicados pelo editor do GitHub.
3. Na página do repositório abra **Actions → Compilar Dungeon Clear (Windows) → Run workflow**.
4. Mantenha `lonelyice_ref` = `v0.1.1` apenas se for a mesma versão do seu LonelyIce. Se for diferente, ajuste para a release que usa.
5. Se aparecer um sinal verde, abra a execução e baixe o artefato `dungeon-clear-windows-experimental`.
6. **Não instale diretamente:** além de compilar, é necessário validar a versão exata do núcleo e testar uma cópia independente do seu servidor. Se a execução falhar, abra a etapa vermelha, copie o erro e peça correção do código.

O ZIP baixado na etapa 5 contém somente o plugin. Não substitua DLLs do servidor e não mexa nas bases SQLite. O addon de interface do jogo é um projeto separado: https://github.com/jrad7/mod-dungeon-clear-addon

## Fontes e notas de adaptação

- LonelyIce: https://github.com/LonelyIceProject/lonelyice — https://lonelyice.org/guides/building-the-core
- Empacotamento de módulos: https://lonelyice.org/guides/wrapping-a-module
- Playerbots para LonelyIce: https://github.com/LonelyIceProject/mod-playerbots
- Dungeon Clear: https://github.com/jrad7/mod-dungeon-clear (AGPL-3.0-or-later)

O workflow fixa o commit upstream no manifesto em cada execução e grava os SHAs em `build-revisions.txt`. Compila contra o core que veio como submódulo da tag. Isso reduz divergências, mas não garante a compatibilidade binária com pacotes lançados anteriormente.

### Limitações conhecidas
- O código do Dungeon Clear usa APIs internas do Playerbots; ele pode exigir patches ao conectar a biblioteca. Uma falha de compilação **não** significa que o seu PC seja incompatível.
- O fonte `mod-dungeon-clear-master` que você baixou não precisa ser enviado ao GitHub, pois o workflow obtém o repositório diretamente.
- A DLL depende do runtime e pode exigir símbolos exportados que a versão do Playerbots não disponibiliza. O pacote não foi testado carregando em um mundo de jogo.
- Execute apenas em repositório GitHub que você controla. Analise dependências externas antes de confiar em qualquer binário.
