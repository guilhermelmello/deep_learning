# WordPiece

O WordPiece é um método de segmentação de textos em sub-palavras (tokens), inicialmente, desenvolvido como parte de um sistema de voz do Google para buscas em japonês e coreano [2]. Como algumas línguas asiáticas possuem poucas, ou nenhuma, forma de separação entre palavras, como espaços em branco, era necessário segmentar os textos. Por conta de suas características, o WordPiece também foi utilizado no sistema de tradução do Google [3] e em modelos como BERT [4] e similares __[[INCLUIR REFERÊNCIAS]]__.

O algoritmo de treinamento do WordPiece foi projetado para criar um inventário (vocabulário) de sub-palavras utilizando apenas os dados de treinamento (data driven) e que, preferencialmente, não produzisse segmentos desconhecidos (OOV -- Out of Vocabulary).

É possível encontrar dois tipos de implementação: top-down e bottom-up. Originalmente, o WordPiece foi implementado seguindo a versão bottom-up, partindo de um inventário de caracteres (alfabeto) e combinando-os para criar novos segmentos. Além da versão original, o WordPiece também pode ser implementado seguindo uma estrutura bottom-up, iniciando o vocabulário com todos segmentos possíveis e eliminando os mais raros. Essa versão top-down foi utilizada pelo BERT e, embora não tenha sido disponibilizada publicamente, pode ser encontrada no TensorFlow [5].

O processamento bottom-up pode ser encontrado na biblioteca Tokenizers, do HuggingFace [1], sendo descrito por [2] da seguinte forma:

1. Construir um vocabulário básico.
2. Utilizando o vocabulário, é construido um modelo de linguagem para calcular a probabilidade de ocorrência de cada par de segmentos.
3. Criar um novo segmento a partir da combinação de outros dois já presentes no vocabulário. A escolha é feita de forma gulosa (greedy), buscando o segmento em que a verossimilhaça (likelihood) dos dados seja máxima.
4. Repita o processo a partir do passo 2 até que o número máximo de segmento seja atingido ou o aumento da verossimilhança seja menor que um certo limiar.

No primeiro passo, o conjunto de caracteres utilizado para inicializar o vocabulário definirá um alfabeto para o modelo e os menores segmentos aceitos durante o processo de segmentação de um texto. Qualquer símbolo fora deste alfabeto resultará em uma segmentação desconhecida (OOV - Out-Of-Vocabulary).

Em [2], o vocabulário é inicializado com os caracteres unicode básicos, sendo cerca de 22 mil para japonês e 11 mil para coreano, tornando rara a ocorrência de segmentos desconhecidos. Porém, em [3], ao utilizar o WordPiece para a tradução de máquina, construir um vocabulário inicial deste tamanho se tornou algo proibitivo, impactando o tempo de treinamento e inferência do modelo. Sendo assim, [3] utilizaram cerca de 500 caracteres para inicializar o vocabulário de línguas ocidentais, impactando consideravelmente a quantidade de segmentos desconhecidos. Uma possibilidade para determinar os caracteres presentes no vocabulário inicial é considerar todos os caracteres presentes no texto de treinamento [1].

Para facilitar o processo de reconstrução de uma sentença são utilizados caracteres especiais apenas para demarcar os limites das palavras. Enquanto [2] utilizam caracteres especiais para demarcar o início e fim de palavras, em [3] são utilizados apenas no início das palavras. Porém, o uso de símbolos especiais (##) para indicar que um segmento é a continuação de uma palavra tem sido a mais adotada, sendo utilizada pelo BERT. Neste caso, segmentos que não são iniciados por esses símbolos especiais indicam o início de uma palavra.

Este algoritmo se torna computacionalmente custoso, durante o segundo passo, ao considerar todas as combinações possíveis de pares [2]. Desta forma, ao calcular a probabilidade de ocorrência de cada par de segmentos, são considerados apenas os pares que ocorrem no dados de treinamento, atualizando apenas a contagem dos segmentos impactados. 

O objetivo no passo 3 é maximizar a verossimilhaça dos dados de treinamento. Isso faz com que o total de segmentos necessários para segmentar os dados de treinamento seja mínimo. Embora [2] não apresentem os detalhes de implementação, maximizar a verossimilhanca é equivalente a escolher o par de segmentos que produzem o maior score [6], sendo definido pela seguinte fórmula [1]:

$score = \frac{f(a, b)}{f(a) \times f(b)}$

em que $a$ e $b$ são dois segmentos presentes no vocabulário, $f(a)$ e $f(b)$ são as frequência com que os segmentos $a$ e $b$ ocorrem nos dados de treinamento, respectivamente, e  $f(a, b)$ é a frequência com que o segmento $a$ precede imediatamente o segmento $b$.

Após a construção do vocabulário, este é utilizado para segmentar novos textos. Para isso, o texto é pré-processado e separado em palavras. Então, é feita uma busca pelo maior segmento que corresponde ao início da palavra. Se a palavra pertence ao vocabulário, então ela resultará em apenas um segmento. O processo é repetido para o segmento restante da palavra podendo chegar em segmentos de apenas um caractere. No caso em que algum caractere não esteja presente no vocabulário, toda a palavra será mapeada para um caractere especial que simboliza um segmento desconhecido ("<unk>", "[UNK]"). Essa segmentação é determinística e pode ser implementada de forma eficiente [7].


# Referências

[1] [HugigngFace - WordPiece Tokenization](https://huggingface.co/course/chapter6/6)

[2] Japanese and Korean voice search

[3] Google's Neural Machine Translation System: Bridging the Gap between Human and Machine Translation

[4] BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding

[5] [Tensorflow - top-down WordPiece](https://www.tensorflow.org/text/guide/subwords_tokenizer?hl=en#optional_the_algorithm)

[6] [HuggingFace - Summary of the Tokenizers - WordPiece](https://huggingface.co/docs/transformers/tokenizer_summary#wordpiece)

[7] Fast WordPiece Tokenization