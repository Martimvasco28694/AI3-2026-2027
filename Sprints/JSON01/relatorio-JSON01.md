# JSON01 · Documentos JSON e Validação com JSON Schemas

**Unidade curricular:** Aplicações para a Internet III · ESTGV
**Aluno:** Martim

## 1. Ficheiros entregues

`disciplinas.json`, `disciplinas.schema.json`, `turma.json`, `turma.schema.json` (UTF-8, aspas duplas retas).

Verificação feita com `python3 -m json.tool` (sintaxe) e `npx ajv-cli validate` (validade). Resultado: `turma.json valid` e `disciplinas.json valid`.

## 2. Variantes de `turma.json` (schema inalterado)

| Variante | Alteração | Resultado | Mensagem |
|---|---|---|---|
| Válida | Retirado o turno TP2 inteiro (`numeroInscritosTp2` e `docenteTp2`) | **Válida** | `turma.json valid` |
| Inválida | Retirado só `docenteTp2` | **Inválida** | `must have property docenteTp2 when property numeroInscritosTp2 is present` (`keyword: 'dependencies'`) |

## 3. Alterações deliberadas (secção 4.4)

| Pedido | Ficheiro | Sintaxe errada, inválida ou válida? | Mensagem obtida | O que a mensagem permitiu (ou não) concluir |
|---|---|---|---|---|
| Retirar a vírgula entre dois pares | turma.json | **Sintaxe errada** | `Expecting ',' delimiter: line 4 column 3 (char 72)` | Indica que falta uma vírgula e a linha/coluna. O documento nem chega ao schema |
| Pôr texto onde é esperado um número | turma.json | **Inválida** | `instancePath: '/totalAlunos'`, `keyword: 'type'`, `message: 'must be integer'` | Sintaxe correta mas tipo errado. Diz onde, que regra falhou e o tipo esperado |
| Retirar uma propriedade obrigatória | turma.json | **Inválida** | `keyword: 'required'`, `missingProperty: 'data'`, `must have required property 'data'` | `instancePath` vazio: o problema está no objeto raiz. Nomeia a propriedade em falta |
| Trocar a ordem de duas propriedades | turma.json | **Válida** | `turma.json valid` | A ordem não importa em JSON, ao contrário do `xsd:sequence` |

## 4. Construções dos esquemas

| Construção | Exemplo | O que ficou a ser impedido | O que continua a ser permitido |
|---|---|---|---|
| `type`/`properties` | `totalAlunos`: integer | `"20"`, `20.5` | Qualquer inteiro absurdo dentro dos limites |
| `required` | `data` | Documento sem `data` | Qualquer valor válido de `data`, em qualquer posição |
| `items` + `minItems`/`maxItems` | `cursos` (2 a 2) | 1 ou 3 cursos | `["tdm","tdm"]` (duplicados) |
| `pattern` | nomes de docentes | `"ana marques"`, `"Inês Sá"` | `"Ines Sa"` |
| `enum` | `data` | `"10-10-2026"` | Apenas as 3 datas |
| `dependencies` | TP1/TP2 | `numeroInscritosTp2` sem `docenteTp2` | `docenteTp2` sozinho |
| `oneOf` | `planoAno1` | Mistura dos dois conjuntos e array vazio (casa com os dois ramos) | Qualquer subconjunto de um só conjunto |

## 5. Reflexão crítica

**1. Ao converter o XML para JSON, perdeu-se informação?**
Perdeu-se a distinção entre atributo e elemento. No JSON, `codigo` e `nome` ficam lado a lado como propriedades do mesmo objeto e já não dá para saber qual era atributo e qual era elemento. Isto não faz diferença para quem só quer ler os dados, mas importa a quem precisa de converter o JSON de volta para XML, porque tem de adivinhar o que devia ser atributo.

**2. A ordem das propriedades: fraqueza ou vantagem?**
Na minha opinião é uma vantagem para quem escreve o programa leitor. O programa vai buscar cada valor pelo nome, por isso não depende da posição e o documento continua válido se a ordem mudar (testei trocando `disciplina` e `codigo` e o resultado foi `valid`). No XML Schema, o `xsd:sequence` obrigava a ordem e qualquer troca dava erro. A desvantagem é que, se alguma vez for importante impor uma ordem, o JSON Schema de base não consegue.

**3. Que valores absurdos o esquema continua a aceitar?**
O `totalAlunos` só tem de estar entre 16 e 24, mas o esquema não o liga aos inscritos nos turnos. Por exemplo, aceita `totalAlunos: 16` com `numeroInscritosTp1: 1` e `numeroInscritosTp2: 2`, o que não faz sentido. Para impedir isto era preciso verificar que `numeroInscritosTp1 + numeroInscritosTp2 = totalAlunos`, e isso o esquema de base não consegue fazer. Teria de ser feito no programa que lê o documento.

**4. O que acontece se se acrescentar uma propriedade nova ao `required`?**
Todos os documentos que já existiam e não têm essa propriedade passam a ser inválidos, mesmo estando corretos até ali. Para evitar o problema, a propriedade nova devia ser acrescentada como opcional (fora do `required`), ou então guardar-se uma nova versão do esquema em vez de mudar o antigo.

**5. Custo de validar vs. custo de não validar**
Validar custa pouco: escrever o esquema e correr um comando. Em troca, os erros aparecem logo à entrada e com a indicação exata de onde estão. Por exemplo, o `ajv` disse-me `must be integer` em `/totalAlunos`. Sem validação, um valor errado só rebenta mais tarde dentro do programa, longe da origem, e quem escreve o leitor tem de escrever código a verificar tudo à mão e a perder tempo a descobrir de onde veio o erro.

## 6. Limitações (CA5)

- Campos extra (ex.: `"observacoes"`) são aceites. Exigiria `additionalProperties` (fora do âmbito).
- `"cursos": ["tdm","tdm"]` é aceite. Exigiria `uniqueItems` (fora do âmbito).
- `totalAlunos = numeroInscritosTp1 + numeroInscritosTp2` não é verificável. Teria de ser feito no programa.
- `"Inês Sá"` falha no pattern porque `[a-z]` não inclui acentos. O erro está no schema, não nos dados.
