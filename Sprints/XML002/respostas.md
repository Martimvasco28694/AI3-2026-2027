# Tarefa XML02 — Validação de documentos XML com Schemas

## 4.1 · Produto

- **ProductID vs Category:** `ProductID` é declarado com `use="required"` (obrigatório); `Category` é declarado sem `use`, por isso é opcional (por omissão, um atributo é opcional).
- **Tipos escolhidos:** `ProductID` → `xsd:int`; `Price` → `xsd:decimal` (um preço pode ter cêntimos, ex.: 120.50); restantes elementos e `Category` → `xsd:string`.
- **Provider:** `minOccurs="0" maxOccurs="3"` (opcional, até três). Como tem elementos dentro (`Name`, `City`), a declaração leva um `complexType` com `sequence` dentro do próprio elemento; o `ProductName` tem apenas um valor, por isso só leva `type="xsd:string"`.

## 4.2 · Aviso

- **Indicador dos contactos:** `xsd:choice` com `minOccurs="1" maxOccurs="2"` (de 1 a 2 contactos no total, cada um email ou telefone).
- **Tipos:** `nome` → `xsd:string`; `numerotrabalho` → `xsd:positiveInteger`; `dataentrega` → `xsd:date`; `email` e `telefone` → `xsd:string`.
- **Variante válida:** troquei "Informa-se o aluno" por "Comunica-se ao docente" no texto livre → `teste2.xml validates`.
- **Variante inválida:** acrescentei um 3.º contacto (`<email>outro@estgv.pt</email>`) → `teste2.xml:8: element email: Schemas validity error : Element 'email': This element is not expected.`

## 4.3 · Verificação

Os quatro ficheiros originais validam (`produto.xml validates`, `aviso.xml validates`).

| Falha | Ficheiro | Mal formado ou inválido? | Mensagem obtida | O que a mensagem permitiu (ou não) concluir |
|---|---|---|---|---|
| Retirar uma etiqueta de fecho (`</Price>`) | produto.xml | **Mal formado** | `parser error : Opening and ending tag mismatch: Price line 7 and Product` | Identifica o elemento (`Price`), mas aponta para a linha 14 (onde o parser deu pela falta) e não para a linha 7 |
| Texto onde é esperado um número | produto.xml | **Inválido** | `Schemas validity error : Element 'Price': 'abc' is not a valid value of the atomic type 'xs:decimal'` | Indica elemento, linha, valor e tipo esperado |
| Trocar a ordem de dois elementos | produto.xml | **Inválido** | `Schemas validity error : Element 'Class': This element is not expected. Expected is ( Price ).` | Diz qual era o elemento esperado naquela posição, o que mostra um problema de ordem |

| Construção | Exemplo | O que ficou a ser exigido | O que continua a ser permitido |
|---|---|---|---|
| `xsd:complexType` | `Product` | Só pode ter os filhos e atributos declarados | Qualquer texto válido do tipo dentro de cada filho |
| `xsd:sequence` | filhos de `Product` | Ordem fixa: `ProductName`, `ProductType`, `Price`, `Class`, `Company`, `Provider` | Qualquer valor do tipo certo (ex.: preço negativo) |
| `xsd:attribute` com `use` | `ProductID` (`required`) | Tem de existir e ser inteiro | `Category` pode faltar ou ter qualquer texto |
| `minOccurs`/`maxOccurs` | `Provider` (0 a 3) | Nunca mais de 3 fabricantes | Zero, um, dois ou três, mesmo repetidos |
| `mixed="true"` (aviso.xsd) | `aviso` | Ordem e multiplicidade dos elementos marcados (`nome`, `numerotrabalho`...) | Texto livre entre os elementos |
## 4.4 · Reflexão crítica

**1. Um esquema resolve o problema das faturas com árvores diferentes? Quem o escreve e quando?**

Resolve em parte. O esquema funciona como um contrato: só é aceite a fatura que cumpre a estrutura combinada, e as outras são rejeitadas mesmo estando bem formadas. Para isso funcionar, tem de ser escrito por quem recebe e processa os documentos (ou por ambas as partes em conjunto) e antes de os documentos serem produzidos. Se for escrito depois, cada pessoa já fez a sua árvore e já não há nada acordado.

**2. Que valores absurdos o `Price` aceita? O que faria falta para os impedir?**

No meu esquema o `Price` é `xsd:decimal`, por isso aceita qualquer número, como `-50`, `0` ou `99999999.99`. O esquema de base só garante que é um número, não que seja um preço razoável. Com os mecanismos desta ficha não consigo impedir isso, por isso a conclusão é que essa verificação tem de ser feita fora do esquema, no programa que lê o documento.

**3. Se amanhã se acrescenta ao esquema um elemento novo obrigatório, o que acontece aos documentos existentes? Que escolha de multiplicidade teria evitado o problema?**

Todos os documentos que já existiam passam a ser inválidos, porque lhes falta um elemento que agora é exigido (um elemento sem `minOccurs` aparece exatamente uma vez). Se o elemento novo tivesse sido declarado com `minOccurs="0"`, ficava opcional: os documentos antigos continuavam válidos e os novos podiam incluí-lo.

**4. Custo de validar vs custo de não validar**

Validar custa escrever e manter o esquema e correr a validação, o que dá pouco trabalho. Não validar sai mais caro: os erros só aparecem dentro do programa, longe da origem (por exemplo, um elemento em falta ou `abc` num preço), e quem escreve o leitor tem de programar defesas para cada caso. Com validação, o erro é detetado à entrada, com uma mensagem que indica o elemento, a linha e o tipo esperado, como vi nas falhas que provoquei, e o programa pode assumir que a estrutura está correta.

**5. Que tipo de documento seria impossível de especificar sem `mixed="true"`? Exemplo diferente da ficha.**

Um documento com texto corrido misturado com elementos marcados, como uma receita em que os ingredientes aparecem no meio das frases (`Misture <ingrediente>200 g de farinha</ingrediente> com o leite`). Sem `mixed="true"`, o texto entre os elementos tornava o documento inválido.