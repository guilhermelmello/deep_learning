# Byte Pair Encoding

O Byte Pair Encoding (BPE) foi originalmente proposto como um algoritmo para compressão de dados [1]. Devido às suas características, foi adaptado para a segmentação de textos [2]. Em [1], dada uma sequência de bytes, o algoritmo de compressão substitui iterativamente a sequência (pares) de bytes mais frequente por um byte que ainda não esteja presente nos dados. Com isso, as sequências de bytes mais frequêntes são removidas, reduzindo o tamanho da sequência de bytes original.

A adaptação realizada por [2] ocorre durante a concatenação de símbolos ao considera o texto como uma sequência de caracteres e não de bytes. Como o objetivo é segmentar o texto, e não a compactação, nenhum caractere é removido durante o processo, mas concatenado para gerar um novo segmento. Para isso, [2] dividem o texto em palavras e cada palavra em uma sequência de caracteres, definindo os segmentos (caracteres) utilizados para inicializar o vocabulário. Em seguida, de forma iterativa, a frequência de pares de caracteres é computada e toda ocorrência do par ('A', 'B') mais frequente é substituida pelo símbolo 'AB', gerando um novo segmento possível. Por questão de eficiência, a contagem de pares é restrita aos limites de cada palavra. O processo de concatenação é repetido até que um número máximo, definido arbitratiamente (hiper-parâmetro), sejam atingido.

Ao considerar palavras como sequências de caracteres Unicode, o BPE permite que os segmentos sejam interpretados como unidades de subpalavras. Porém, assim como o WordPiece, este método permite a produção de segmentos desconhecidos pelo modelo (OOV - Out-Of-Vocabulary). Para evitar OOV durante a segmentação e não ter que utilizar o inventário de caracteres Unicode com mais de 130 mil símbolos para inicializar o vocabulário, [3] utilizaram a estratégia diferente e consideram as palavras como sequências de bytes. Ao utilizar o nível de bytes a interpretação dos segmentos pode ser prejudicada, dado que caracteres Unicodes utilizam mais de 1 byte para sua codificação e podem ser dividos em segmentos diferentes. Porém, o vocabulário inicial fica limitado aos 256 bytes possíveis, garantiram que nenhum segmento desconhecido possa ser produzido. O uso de BPE em nível de bytes é conhecido como byte-level BPE e foi utilizado por outros modelos como RoBERTa [4].

Na prática, além do nível de caracteres e bytes, outras diferenças ocorrem ao considerar a aplicação em tarefas de NLU e NLG. No caso do GPT-2 [3], o foco foi a geração de textos, sendo necessário incluir apenas o símbolo especial para representar o fim de um texto. Como o modelo utilizou o nível de bytes, não foi necessário realizar um pré-processamento do texto de entrada. Para modelos de NLU, como foi o caso do modelo RoBERTa [4], há a necessidade de incluir símbolos para classificação, separação e preenchimento de sentenças, além de um símbolo para treinamento do modelo em MLM. Para o caso de BPE em nível de caracteres, ainda é necessário incluir um símbolo para segmentos desconhecidos, sendo necessário realizar alguma padronização do texto de entrada.


- gpt-2: merge é feito entre caracteres unicode do mesma categoria, com excessão de espaços.

# Referências

[1] A New Algorithm for Data Compression

[2] Neural Machine Translation of Rare Words with Subword Units

[3] Language Models are Unsupervised Multitask Learners (GPT2)

[4] RoBERTa: A Robustly Optimized BERT Pretraining Approach
