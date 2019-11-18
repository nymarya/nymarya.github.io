---
layout: post
title: "CNNS: A covolução"
image: "https://i.stack.imgur.com/VC1TX.png"
categories:
 - Edge Case
tags:
 - deep learning
 - CNN
---

Este é o primeiro post de uma série sobre deep learning, especialmente Redes Neurais Convolucionais (em inglês: Convolutional Neural Networks, ou CNNs) que consiste de dois post teóricos e um mais prático, discorrendo sobre um modelo específico. Vamos começar com o C da sigla em inglês, CNN.

**Convolução** é um termo usado na área da matemática para descrever a operação sobre duas funções que produzem uma terceira função expressando como a primeira é modificada pela outra. Eu sempre me recordo de uma aula na qual o professor explicou esse conceito de uma maneira bem simples. Imagine que você está entre dois palcos durante um festival de música: se você se mover em direção a um ou outro, ouvirá melhor alguma das músicas. Se ficar no meio, os sinais ficam muito misturados. Esta confusão de ruídos é a convolução em ação.

## Sobre imagens

<div align="center">
<img src="https://media.giphy.com/media/8b5gDEqjO5BKM/giphy.gif">
</div>

Primeiro, o básico de imagens. Para o computador, uma imagem não passa de de uma matrix de inteiros. Quando a imagem é em tons de cinza, cada número entre 0 e 255 representa a luminosidade de um pixel. Existem duas maneiras de representar figuras RGB. A primeira é como um grupo de 3 inteiros que compoem um pixel, com cada elemento do trio representando o valor de vermelho, verde e azul, respectivamente. Na outra forma, a imagem possui dimensões mxnx3, tendo uma camada para cada cor.

Em relação a processamento de imagens, convolucionar significa executar várias multiplicações de matrizes entre a matrix que representa a imagem e a  [máscara](https://en.wikipedia.org/wiki/Kernel_(image_processing). Deste modo, nós podemos aplicar filtros, seja para detecção de bordas, efeitos de blurring ou de sharpening.

## Tipos de convolução de imagem

Embora uma multiplicação de matrizes convencional requeira que o número de colunas da primeira matrix seja igual ao número de linhas da segunda matriz, o processo de convolução não possui este tipo de condição. As dimensões do kernel não precisam bater com as da imagem, visto que a convolução não equivale a um produto convecional. Ao invés disso, a operação é realizada sucessivamente sobre a figura, como se fosse uma janela deslizante movendo sobre a imagem.

Considerando que os formatos não estão adequados, existem tipos de convolução cujos resultos tem dimensões diferentes. O método [convolve](https://docs.scipy.org/doc/scipy/reference/generated/scipy.signal.convolve2d.html) do módulo scipy possui três modos: *full*, no qual os sinais não se sobrepõe completamente nas bordas da operação e podem acontecer efeitos nas bordas; *same*, no qual o resultado tem o mesmo tamanho da imagem; e *valid*, onde o product consiste apenas dos elementos que não dependem de zero-padding.

## Sobre matemática

<div align="center">
<img src="https://media.giphy.com/media/26xBI73gWquCBBCDe/giphy.gif">
</div>

A operação, como dito anteriormente, é muito similiar à multiplicação de matrizes comum. As diferenças começãm com o tamanho da matriz final. No modo *full*, a matriz resultante C tem tamanho *m* que combina o tamanho da matriz kernel K e a imagem B, conforma a equação abaixo:

$$
m = |K| + |B| - 1
$$

Os valores de $$C$$ são obtidos ao multiplicar K e B.

$$
C(i, j) = \sum_{p=0}^{i+1} \sum_{q=0}^{j+1} K(p,q) B(i - p + 1, j - q + 1)
$$

onde $$i$$ e $$j$$ são posições em $$C$$, e a notação $$M(a, b)$$ denota a elemento na a-ésima linha e b-ésima coluna.

## Convolucionar para aprender

A inspiração de usar a ideia de convolução para construir um algoritmo de aprendizado de máquina vem de um mecanismo biológico encontrado no cérebro de mamíferos. O córtex visual contem grupos de neurônios que são modulados por **campos receptivos**, que "[podem provocar reações neurais quando estimuladas](https://en.wikipedia.org/wiki/Receptive_field)". Campos receptivos podem se sobrepor e repetir entre células visiznhas, de forma que atuam como filtros locais sobre o que recebem.

O [estudo](https://doi.org/10.1113%2Fjphysiol.1968.sp008455), que traz informações sobre a região, divide as células em dois tipos: *simples*, que reagem a padrões específicos de seu campo receptivo; e *complexas*, que tem campos receptivos maiores e não são sensíveis à exata posição de um padrão na área.

## Discussão

É importante (além de interessante! ) aprender sobre todos os passos no processo do aprendizado computacional, e é especialmente poderoso enteder a razão pela qual o modelo foi criado de certa maneira. Nos próximos posts, vamos continuar a ver teoria e depois investigar e treinar um modelo de deep learning.
Até mais!

## Para saber mais & referências

[Processamento de imagens com OpenCV](https://docs.opencv.org/master/d2/d96/tutorial_py_table_of_contents_imgproc.html)

[CS123N](https://cs231n.github.io/convolutional-networks/)

[Activations of DCNNs are aligned with gamma band activty of human visual cortex](https://www.nature.com/articles/s42003-018-0110-y)

[Brain and the visual perception](https://books.google.com/books?id=8YrxWojxUA4C&pg=PA106)

[Receptive fields of single neurones in the cat's striate cortex](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC1363130)
