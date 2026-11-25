# Engenharia de prompts e expressões regulares em Python

## Informações gerais

- **Público-alvo**: estudantes da disciplina de **Sistemas Operacionais** do curso de [TADS](https://diatinf.ifrn.edu.br/cursos/tecnologia-em-analise-e-desenvolvimento-de-sistemas/) na [DIATINF](https://diatinf.ifrn.edu.br/).
- **Professor**: [L. A. Minora](https://github.com/leonardo-minora/).
- **Duração**: 90 minutos.
- **Objetivo geral**: praticar engenharia de prompts para criar, testar e revisar expressões regulares em Python, compreendendo seus limites e riscos.
- **Objetivos de aprendizagem**:
  - Aplicar o ciclo iterativo **prompt → teste → diagnóstico → novo prompt**.
  - Escrever prompts de depuração que incluam entrada, resultado obtido e resultado esperado.
  - Avaliar regex geradas por IA quanto à corretude, legibilidade, casos de borda e desempenho.
  - Usar quantificadores não gulosos, *lookahead*, *lookbehind* e `re.VERBOSE`.
  - Reconhecer situações em que uma regex não é a ferramenta adequada.

---

## Conceito central

**Refinamento iterativo e verificação.** Uma resposta gerada por IA é uma **hipótese**, não uma solução automaticamente correta. O estudante deve testar a regex, identificar um caso que falhou e fornecer à IA evidências concretas: a entrada, a saída obtida e a saída esperada. Depois, deve testar novamente a proposta revisada.

Regex também têm limites. Por exemplo, não são uma ferramenta adequada para analisar estruturas aninhadas arbitrárias, como HTML com tags aninhadas; nesse caso, prefira um parser específico.

## Resumo

Prompts melhoram quando incorporam evidências reais e critérios claros. Regex geradas por IA devem ser revisadas antes de serem usadas: precisam funcionar nos casos previstos, ser compreensíveis, evitar falsos positivos e ter desempenho aceitável. Dados pessoais reais não devem ser enviados a serviços de IA.

---

## Roteiro da aula (90 minutos)

| Etapa | Tempo | Atividade |
|---|---:|---|
| Exposição dialogada | 30 min | Engenharia de prompts, ciclo iterativo, fundamentos de regex e limites/riscos. |
| Prática guiada | 40 min | Em duplas, solicitar uma regex, criar testes, diagnosticar falhas e iterar. |
| Trabalho livre e socialização | 20 min | Desafio de extensão, revisão por pares e registro das decisões. |

### 1. Exposição dialogada — 30 minutos

1. **Abertura e problema (5 min)**: apresentar uma tarefa simples de extração de códigos e perguntar como saber se a resposta da IA está correta.
2. **Prompt com contexto e critérios (8 min)**: comparar um pedido vago (“faça uma regex”) com um pedido que especifica linguagem, formato, exemplos positivos e negativos e restrições.
3. **Ciclo de depuração (7 min)**: mostrar como comunicar um contraexemplo com entrada, saída atual e resultado esperado.
4. **Recursos e limites (7 min)**: demonstrar guloso/não guloso, lookarounds e `re.VERBOSE`; discutir falsos positivos, backtracking e quando escolher outra ferramenta.
5. **Ética e transição à prática (3 min)**: não compartilhar dados pessoais reais e informar/citar o uso de IA conforme as orientações da disciplina.

### 2. Prática guiada — 40 minutos

Trabalhem em duplas. Uma pessoa conduz os prompts e a outra atua como revisora, conferindo os testes e questionando os casos de borda. Troquem de papel na segunda iteração.

#### Desafio: extrair códigos de chamados

Considere que um sistema registra identificadores no formato `CH-` seguido de exatamente quatro algarismos ASCII, por exemplo `CH-2048`. A tarefa é encontrar identificadores válidos em um texto sem aceitar correspondências parciais dentro de sequências maiores.

**Etapa A — Especificar e pedir uma hipótese (8 min)**

Antes de consultar a IA, escreva os requisitos e pelo menos três exemplos válidos e três inválidos. Em seguida, use um prompt como este:

> Você é um assistente de programação que explica suas decisões. Em Python, proponha uma expressão regular para localizar códigos no formato `CH-` seguido de exatamente quatro algarismos ASCII. Não aceite correspondências parciais quando houver letras ou algarismos imediatamente antes ou depois do código. Considere estes exemplos: positivos `CH-2048`, `(CH-0007)`; negativos `XCH-2048`, `CH-20480`, `CH-20A8`. Explique cada parte da regex, apresente uma versão legível com `re.VERBOSE` e sugira pelo menos três casos de borda. Não afirme que a solução está correta sem testá-la.

**Etapa B — Testar a hipótese (10 min)**

Rode o código localmente. Não use dados pessoais ou registros reais. Compare todos os resultados com o comportamento esperado e acrescente casos como texto vazio, código no começo/fim, pontuação adjacente, letras acentuadas e códigos repetidos.

Exemplo de estrutura para os testes:

```python
import re

PADRAO = re.compile(r"""
    (?<![A-Za-z0-9])  # não começar no meio de uma sequência alfanumérica
    CH-
    [0-9]{4}          # quatro algarismos ASCII
    (?![A-Za-z0-9])   # não terminar no meio de uma sequência alfanumérica
""", re.VERBOSE)

casos = {
    "Chamado CH-2048 aberto": ["CH-2048"],
    "(CH-0007)": ["CH-0007"],
    "XCH-2048": [],
    "CH-20480": [],
    "CH-20A8": [],
    "": [],
}

for texto, esperado in casos.items():
    obtido = PADRAO.findall(texto)
    assert obtido == esperado, (
        f"Entrada: {texto!r}; obtido: {obtido}; esperado: {esperado}"
    )

print("Todos os testes passaram.")
```

Os lookarounds `(?<!...)` e `(?!...)` verificam o caractere anterior e o seguinte sem incluí-los na correspondência. Aqui, `[0-9]` deixa explícito que o formato aceita algarismos ASCII. Se a especificação permitir outros algarismos Unicode ou definir outros separadores, isso deve ser decidido e testado explicitamente.

**Etapa C — Diagnosticar e refinar (12 min)**

Se um teste falhar, não peça apenas “corrija a regex”. Envie à IA um relato reproduzível:

> A regex anterior foi `...` e o código de teste foi `...`. Para a entrada `...`, obtive `...`, mas esperava `...`. O requisito é `...`. Explique a causa provável, proponha a menor alteração necessária e atualize a lista de testes, incluindo um caso que confirme que a correção não quebrou o comportamento anterior.

Registre cada iteração: prompt, resposta, teste que falhou, diagnóstico e alteração feita. Confira se a mudança resolveu o contraexemplo sem introduzir falsos positivos.

**Etapa D — Revisar legibilidade e riscos (10 min)**

Peça à IA uma explicação por partes e uma forma legível com `re.VERBOSE`. Depois, a dupla deve revisar a solução sem depender da explicação da IA:

- Os limites do identificador estão especificados com clareza?
- Os exemplos negativos são rejeitados?
- Há uma forma mais simples de resolver com `str` methods?
- A regex pode ser lida e mantida por outra pessoa?
- O padrão e o tamanho da entrada são controlados? Há motivo para se preocupar com backtracking excessivo?

### 3. Trabalho livre e socialização — 20 minutos

Escolham uma extensão: aceitar mais de um prefixo, extrair todos os códigos de um texto, ou comparar uma solução com regex a uma solução com `str` methods. Criem pelo menos dois testes adicionais e façam revisão cruzada com outra dupla. Cada dupla apresenta em até um minuto uma falha encontrada, como a corrigiu e uma limitação que permaneceu.

---

## Conteúdo de referência

### Como escrever um bom prompt de programação

Um prompt útil fornece:

1. **Contexto**: linguagem, biblioteca e finalidade.
2. **Requisitos observáveis**: o que deve ser aceito e rejeitado.
3. **Exemplos**: entradas positivas e negativas.
4. **Restrições**: por exemplo, evitar correspondências parciais ou não usar bibliotecas externas.
5. **Formato da resposta**: regex, explicação, testes e limitações.
6. **Pedido de incerteza**: solicitar que a IA indique ambiguidades em vez de inventar requisitos.

A resposta da IA não substitui testes. Uma regex pode passar nos exemplos fornecidos e ainda falhar em casos de borda não considerados.

### Guloso e não guloso

Quantificadores como `*` e `+` são gulosos por padrão: tentam consumir o máximo possível, voltando atrás quando necessário. As versões não gulosas `*?` e `+?` tentam consumir o mínimo possível.

```python
import re

texto = "<a>primeiro</a> e <a>segundo</a>"
print(re.findall(r"<a>.*</a>", texto))   # uma correspondência ampla
print(re.findall(r"<a>.*?</a>", texto)) # duas correspondências menores
```

O quantificador não guloso não transforma regex em parser de HTML: atributos, comentários, conteúdo especial e aninhamento tornam essa abordagem frágil.

### Lookahead e lookbehind

- `(?=...)`: *lookahead* positivo; exige que algo venha a seguir, sem consumir esse trecho.
- `(?!...)`: *lookahead* negativo; exige que algo não venha a seguir.
- `(?<=...)`: *lookbehind* positivo; exige uma condição imediatamente anterior, sem incluí-la na correspondência.
- `(?<!...)`: *lookbehind* negativo.

No módulo `re` do Python, lookbehind deve ter largura fixa. Verifique a documentação e teste a expressão no ambiente da atividade antes de depender de um padrão mais complexo.

### `re.VERBOSE`

Com `re.VERBOSE` (ou `re.X`), espaços e comentários podem tornar uma expressão longa mais fácil de revisar. Espaços literais precisam ser escapados ou representados explicitamente quando forem significativos. Prefira nomes descritivos para padrões e testes automatizados para explicar o comportamento pretendido.

### Riscos e limites

- **Falsos positivos e falsos negativos**: exemplos insuficientes deixam lacunas na especificação.
- **Backtracking excessivo (ReDoS)**: certos padrões ambíguos podem ter desempenho muito ruim em entradas cuidadosamente construídas. Evite padrões excessivamente complexos, limite o tamanho das entradas quando apropriado, use testes e medições seguros e revise padrões antes da produção. Não execute demonstrações de padrões potencialmente catastróficos em serviços ou sistemas compartilhados.
- **Validação semântica**: uma regex pode verificar o formato superficial de um CPF, mas não deve ser tratada como validação completa; dígitos verificadores exigem lógica própria, além de regras de negócio.
- **Estruturas aninhadas**: HTML, XML e linguagens com aninhamento devem ser analisados com parsers apropriados, não com uma regex improvisada.
- **Privacidade**: substitua dados reais por exemplos sintéticos antes de consultar ferramentas de IA. Não envie senhas, tokens, dados pessoais ou informações internas.
- **Autoria e transparência**: siga as regras da disciplina e declare/cite o uso de IA quando solicitado. Você continua responsável por compreender, testar e apresentar o código.

### Quando não usar regex

Prefira ferramentas específicas quando forem mais simples e seguras:

- `str.startswith`, `str.endswith`, `str.split` ou `str.replace` para operações simples com texto;
- `json` para ler e validar a estrutura de JSON;
- `datetime` para interpretar datas e horários;
- parsers de HTML/XML para documentos estruturados;
- código de validação semântica para regras que dependem de cálculos ou contexto.

---

## Avaliação formativa

A dupla deve entregar um registro breve contendo:

1. O requisito em linguagem natural e os exemplos positivos/negativos.
2. O prompt inicial e a regex sugerida.
3. Os testes executados e ao menos um contraexemplo, real ou criado para desafiar a hipótese.
4. O diagnóstico e o prompt de refinamento.
5. A versão final, sua explicação e uma limitação ou alternativa à regex.
6. Uma declaração sobre o uso de IA, conforme as orientações da disciplina.

**Critérios de sucesso:** testes reproduzíveis; correção nos casos especificados; explicação compreensível; consideração de casos de borda, privacidade e desempenho; e capacidade de justificar quando a regex é ou não apropriada. O objetivo não é obter a regex mais curta, mas demonstrar um processo de verificação responsável.
