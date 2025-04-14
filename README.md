# Autores e Artigos do prosa.dev

Este repositório contém os autores e artigos do prosa.dev, um blog que trata sobre tecnologia e coisas da vida sempre de maneira leve e informal.

## Autores

O arquivo de autores é um YAML com a seguinte estrutura:

```yaml
username: pancho
displayName: "Pancho"
displayPicture: "https://url.com.br/author/avatar.jpg"
bio: "Upa pequena biografia sobre o autor."
links:
    bitbucket: "https://bitbucket.org/author",
    discord: "https://discord.com/author",
    facebook: "https://facebook.com/author",
    github: "https://github.com/author",
    gitlab: "https://gitlab.com/author",
    instagram: "https://instagram.com/author",
    linkedin: "https://linkedin.com/author",
    reddit: "https://reddit.com/author",
    telegram: "https://t.me/author",
    tiktok: "https://tiktok.com/@author",
    website: "https://prosa.dev",
    x: "https://x.com/author",
    youtube: "https://youtube.com/author"
```

Este arquivo deve ser nomeado com o nome de usuário do autor, seguido da extensão `.yaml`. Por exemplo, o arquivo do autor Pancho deve ser nomeado como `pancho.yaml`.
Os links são opcionais, mas é recomendável que pelo menos um deles seja preenchido.

O local dos arquivos de autores é `/authors/`.

## Artigos

Os artigos são arquivos Markdown com front-matter e seguem a seguinte estrutura:

```markdown
---
title: Título do Artigo
cover: images/nome_da_imagem_de_capa.jpg
author: username_do_autor
categories:
    - Categoria 1
    - Categoria 2
    - Categoria 3
tags:
    - Tag 1
    - Tag 2
    - Tag 3
excerpt: |
    Um pequeno resumo do artigo, que pode ser usado como descrição.
    O tamanho máximo é de 500 caracteres.
---

Conteúdo do artigo
```

Artigos devem conter extensão `.md` e deverão ser criados em `/articles/username_do_autor/slug_do_artigo/` .

Um subdiretório `images` poderá ser criado para acomodar as imagens do artigo, e este será processado pelo sistema no momento em que o artigo for publicado.

## Categorias do Artigo

As categorias podem ser adicionadas de forma livre, mas é recomendável que sigam o assunto tratado no artigo.

Elas servem para organizar os artigos e listar artigos recomendados que tenham a ver com o assunto do artigo sendo lido.

## Tags do Artigo

As tags são palavras-chave que auxiliam no SEO (Search Engine Optimization) do artigo, e são utilizadas para categorizar o artigo de forma mais específica. Também podem ser criadas de forma livre, e não deve ser usado espaço, letras maiúsculas, hífen ou caracteres especiais. O ideal é usar apenas letras minúsculas e o caractere `_` (underscore) para separar as palavras. Por exemplo: `minha_tag_de_exemplo`.
