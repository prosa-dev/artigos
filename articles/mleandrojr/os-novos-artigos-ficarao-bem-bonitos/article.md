---
title: Os Novos Artigos Ficarão Bem Bonitos
cover: images/cover.jpg
author: mleandrojr
categories:
  - JavaScript
  - Front-End
  - Next.js
  - React
tags:
  - javascript
  - react
  - nextjs
  - frontend
excerpt: |
    Neste artigo contarei como ficou o sistema com o parser de markdown e por que acho o syntax highlighting importante neste site.
---

Fala, galerinha!

No artigo passado ([que você pode ler clicando aqui](/article/preciso-escrever-mais-aqui)), eu comentei que estava pensando em montar um processo de CI/CD para este site.

Antes disso, porém, precisei desenvolver todo o sistema que adicionará um artigo ao site, pois conforme contei naquele artigo, o processo era manual.

Depois de um fim de semana escrevendo código, montei um sistema que receberá os artigos e adicionará ao banco de dados, e isso acontece de forma bastante simples.

# Como o sistema funciona?

Receber atualizações de arquivos e adicionar ao banco de dados é uma tarefa muito simples. Muito mais do que você possa imaginar, e a seguir falarei sobre todo o processo.

## Autores

Antes de mais nada, artigos são escritos por autores, então precisamos adicioná-los também.

O formato definido para criação de autores foi o `YAML`.
Este formato comporta todas as informações que um autor deve ter, é rápido de escrever e de fácil leitura.

```yaml
username: mleandrojr
displayName: "Marcos Leandro"
displayPicture: "https://avatars.githubusercontent.com/u/2152874"
bio: "Desenvolvedor Full Stack há bastante tempo, gamer, motoca e viajante.\n
    Gosto muito de tecnologia, mas também de antiguidades. Tenho vários projetos publicados (como este aqui) e vários outros
    em planejamento ou desenvolvimento. Talvez mais alguns deles vejam a luz do dia, mas não prometo nada."
links:
    github: "https://github.com/mleandrojr"
    youtube: "https://youtube.com/@aqueledev"
    telegram: "https://t.me/aqueledev"
    linkedin: "https://www.linkedin.com/in/mleandrojr/"
    instagram: "https://instagram.com/asqueledev"
    tiktok: "https://tiktok.com/@aqueledev"
```

Existe um cron que varre o diretório de autores e os adicionada ao banco de dados. Se o autor já existir, então os dados são atualizados.

Não existe exclusão de autores desta forma. Isso ainda tem que ser feito de forma manual.

Sobre o processo, falaremos mais tarde.

## Artigos

Os artigos estão em formato `markdown`, e como artigos precisam de cabeçalhos informando tags, categorias, título e outras informações, resolvi trabalhar com `front matter`.

Ele consiste em um cabeçalho `YAML` no início do `markdown`:

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

Escrever em `markdown` é fácil e dará várias possibilidades para enriquecer os artigos, e isso se deve a uma particularidade que falarei mais à frente.

## O Processo

Tudo começa com um [pull request no repositório de artigos](https://github.com/prosa-dev/artigos) .

Depois de revisado e aceito, eu faço um `pull` do `origin` lá no servidor e ele se encarregará do resto.

Existem duas `crons` que rodam a cada 5 minutos, uma que varre os diretórios de autores e outra que varre os diretórios de artigos.

Parece um processo pesado, mas na verdade não é tanto assim.

Os arquivos que já foram processados são pulados no processo, e o sistema processa somente os arquivos novos.

Como isso acontece?

Estou usando a classe `RecursiveDirectoryIterator` do PHP, e além de iterar recursivamente sobre o diretório selecionado, ele também dá acesso a algumas informações dos arquivos. Uma delas é muito útil para isso: o `mtime`.

```php
$mtime = $file->getMTime();
```

O `mtime` nada mais é do que o timestamp da última modificação do arquivo, e ele é útil porque é ele quem define se o arquivo deve ser processado ou não.

No começo do processo eu faço a leitura de um arquivo de índice (`authorsidx` para autores e `articlesidx` para artigos) e salvo em uma variável `$lastModifiedTime`.

Se o arquivo não existir ou se estiver vazio, defino a esta variável o valor padrão `0`.

Depois, itero sobre o `RecursiveDirectoryIterator` e verifico se o `mtime` do arquivo é maior do que o valor definido em `$lastModifiedTime`.

Se for, então o arquivo é processado e o novo `mtime` salvo em uma variável temporária.

No final do processo, verifico qual é o maior `mtime` salvo nesta variável temporária e gravo no arquivo de índice.

## Formato

Pensei bastante no formato que eu usaria para salvar os artigos no banco de dados.
A dúvida estava entre salvar o markdown, ou salvar o artigo já convertido em HTML.

Cada um tem suas vantagens e desvantagens, mas o que me fez avaliar a situação foi o seguinte:

Convertendo o artigo em HTML antes de salvar no banco de dados, eu onero um pouco mais o php-fpm no momento do processamento, porém alivio o servidor do next.js que não precisará fazer a conversão em cada requisição. Os tempos de resposta durante a navegação são menores.

Esta abordagem entregará o HTML pronto, e se houver alguma alteração de layout, precisarei editar manualmente os artigos já salvos, ou criar um script para fazer isso.

Por outro lado, se eu salvar o artigo em markdown, o servidor do next.js terá que fazer a conversão em cada requisição, o que pode atrasar um pouco a resposta, mas me dará mais flexibilidade para alterações futuras.

Depois de testes, optei por salvar o artigo em markdown, e deixei o servidor do next.js se encarregar de convertê-lo em HTML mesmo. O aumento no tempo de resposta foi quase que imperceptível e futuras alterações de layout serão aplicadas em todos os artigos automaticamente.

# Conversão para HTML e Syntax Highlighting

Apesar de mais informal, este é o site de um dev, e como tal, provavelmente escreverei muitos códigos aqui.

Ter uma visualização bem bonita e brilhante é bacana, então resolvi adicionar um `syntax highlighting` ao site.

Resolvi trabalhar com o `unified()` para gerenciar o pipe e os plugins `remark-*` para fazer o processamento do artigo. Depois de processado, uso o `rehype-prism` para fazer o `syntax highlighting`.

```bash
[mleandrojr@localhost ~]$ npm install remark-gfm remark-parse remark-rehype rehype-prism rehype-stringify
```

Depois de instaladas as bibliotecas, precisamos importá-las.

```javascript
import { unified } from "unified";
import remarkGfm from "remark-gfm";
import remarkParse from "remark-parse";
import remarkRehype from "remark-rehype";
import rehypePrism from "rehype-prism";
import rehypeStringify from "rehype-stringify";
```

Parece bastante coisa, mas é porque cada uma delas faz uma coisa que deixará o artigo mais bacana de se ver.

Tenho uma variável `articleContent` com o conteúdo bruto do artigo, e começo o processamento assim:

```javascript
const processedContent = unified()
	.use(remarkParse)
```

O `remarkParse` é o responsável por fazer a leitura do conteúdo e transformá-lo em um `tree` que o `unified` entende.

Em seguida, uso o `remarkGfm`, que é o responsável por adicionar o suporte a tabelas, listas e mais uma porrada de coisa do `GitHub Flavored Markdown`.

Isso enriquece bastante o artigo, e se você quiser saber mais sobre o `GFM`, você pode ler [aqui](https://github.github.com/gfm/).

Lembra que eu falei acima sobre as possibilidades? Então. Elas estão todas aqui.

```javascript
const processedContent = unified()
	.use(remarkParse)
	.use(remarkGfm)
```

Depois disso, uso o `remarkRehype`, que é o responsável por transformar o `tree` do `remark` em um `tree` que o `rehype` entende.

```javascript {4}
const processedContent = unified()
	.use(remarkParse)
	.use(remarkGfm)
	.use(remarkRehype)
```

Uso o `rehypePrism`, que é o responsável por finalmente fazer o `syntax highlighting` do código, adicionando ainda os números das linhas ao código.

```javascript {5}
const processedContent = unified()
	.use(remarkParse)
	.use(remarkGfm)
	.use(remarkRehype)
	.use(rehypePrism, { plugins: ["line-numbers"] })
```

E por último, uso o `rehypeStringify`, que é o responsável por transformar o `tree` do `rehype` em uma `string` com um `HTML`.

```javascript {6}
const processedContent = unified()
	.use(remarkParse)
	.use(remarkGfm)
	.use(remarkRehype)
	.use(rehypePrism, { plugins: ["line-numbers"] })
	.use(rehypeStringify)
```
Agora, é só chamar o `processSync()` para processar tudo isso e retornar o artigo pronto.

```javascript {7}
const processedContent = unified()
	.use(remarkParse)
	.use(remarkGfm)
	.use(remarkRehype)
	.use(rehypePrism, { plugins: ["line-numbers"] })
	.use(rehypeStringify)
	.processSync(articleContent);
```
# E agora?
Agora preciso finalmente montar o processo de CI/CD.
Será feito com GitHub Actions mesmo, executando sempre que um pull request for aprovado.
Este artigo provavelmente será publicado através dele, pois será um excelente teste para a rotina.
Então, se você chegou até aqui, é porque todo o processo deu certo.

Pode ser que eventualmente eu conte sobre a parte que roda no GitHub. (=
