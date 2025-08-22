---
title: Desmistificando uma Bomba
cover: images/cover.jpg
author: mleandrojr
categories:
  - Linux
  - Shell
  - Bash
tags:
  - linux
  - bomba
  - shell
  - bash
excerpt: |
    Neste artigo contarei como ficou o sistema com o parser de markdown e por que acho o syntax highlighting importante neste site.
---

Você certamente já deve ter visto em algum lugar isso aqui:

```bash
:(){ :|:& };:
```
E quando viu, provavelmente leu que se tratava de uma bomba.

Mas o que diabos é isso? E como exatamente isso funciona?

# Definição
O código acima pode ser chamado de fork bomb, ou bomba de fork. Fork é o termo usado para descrever a criação de um novo processo em sistemas Unix-like através de um processo principal.

# Funcionamento
A bomba de fork funciona criando um processo que se replica indefinidamente, consumindo todos os recursos do sistema, como CPU e memória, até que o sistema fique inutilizável.

Vamos analisar o código em partes:

1. `:()` - Define uma função chamada `:`. Em bash, você pode nomear funções com quase qualquer caractere, incluindo `:`. Poderia ser utilizado qualquer outro nome, mas o uso de `:` é uma forma de tornar o código mais obscuro.

2. `{ ... }` - É o delimitador de bloco de código da função. Tudo que estiver dentro dessas chaves será executado quando a função for chamada. Quem programa já está acostumado com esses delimitadores.

3. `:|:` - Dentro do bloco, a função `:` chama a si mesma duas vezes, criando dois processos filhos. O operador `|` é o pipe, que normalmente é usado para redirecionar a saída de um comando para a entrada de outro. Com isso, direcionamos a saída da primeira chamada à entrada da segunda.

4. `&` - Coloca o processo em segundo plano, permitindo que o shell continue a executar outros comandos. Isso é crucial para a bomba de fork, pois permite que os processos se multipliquem rapidamente.

5. `;` - Indica o fim do comando dentro do bloco da função.

6. `:` - Finalmente, a função é chamada pela primeira vez, iniciando o ciclo de replicação.

Na prática, quando a função é chamada, ela cria dois processos filhos, cada um dos quais também chama a função `:` novamente, criando mais dois processos filhos, e assim por diante. Isso leva a uma explosão exponencial no número de processos em execução.

## Consequências
O resultado é que o sistema rapidamente fica sobrecarregado com processos, consumindo todos os recursos disponíveis, o que pode levar a uma falha do sistema ou a necessidade de um reinício forçado.

## Mas isso realmente funciona?
Sim, funciona. Mas não se preocupe, a maioria dos sistemas modernos têm mecanismos de proteção contra esse tipo de ataque, como limites no número de processos que um usuário pode criar.

Este limite pode ser verificado com o comando:

```bash
[prosa@dev ~]$ ulimit -u
```

E você pode alterá-lo temporariamente com:

```bash
[prosa@dev ~]$ ulimit -u <novo_limite>
```

Ou permanentemente editando o arquivo `/etc/security/limits.conf` e adicionando uma linha como:

```
<username> <type> nproc <limit>
```

## Conclusão
A fork bomb é um exemplo clássico de como um código pequeno e aparentemente simples pode ter consequências devastadoras para um sistema.

Embora seja interessante conhecer esse tipo de código, é importante lembrar que executá-lo em um sistema real pode causar danos significativos e deve ser evitado a todo custo.

Espero que este artigo tenha ajudado a desmistificar o que é uma fork bomb e como ela funciona.

Até a próxima!
