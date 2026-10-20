# Documentação do Vision.AI

Este diretório é o **espaço de documentação** do projeto de instalação. Aqui
ficam notas de operação e materiais que acompanham o instalador, separados do
`README.md` da raiz (que é o guia de instalação).

## Onde fica o manual completo

O **manual de uso** (em português, inglês e espanhol) é um site VitePress cuja
**fonte** vive no repositório do código, em
[`bortolottic/vision.ai`](https://github.com/bortolottic/vision.ai), na pasta
`docs/`. O workflow `.github/workflows/docs.yml` daquele repositório constrói o
site e o publica no branch **`gh-pages`** deste repositório a cada merge na `main`
e a cada tag de release.

- **Online:** https://bortolottic.github.io/vision.ai.installer/
- **Offline:** a mesma documentação é servida pela própria instalação em `/docs/`.

> Para habilitar o site na primeira vez: **Settings → Pages → Deploy from a
> branch → `gh-pages` / (root)** neste repositório, depois que o workflow de docs
> rodar pela primeira vez e criar o branch.

## O que colocar aqui

- Notas de versão específicas da operação (migrações manuais, mudanças de porta).
- Runbooks internos (backup/restore, rotação de certificado, troca de segredos).
- Qualquer material de instalação que não seja o manual de uso da aplicação.
