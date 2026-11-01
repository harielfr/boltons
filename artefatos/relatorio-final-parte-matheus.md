# Relatório Final – parte de Matheus dos Santos (boltons)

Nov 1, 2026 · @Matheus

## Como usar este documento

Este texto traz a parte de Matheus no [modelo do relatório final](https://docs.google.com/document/d/1jxzD5XHE7IMSVOERLYlLdWUtRYZY5jVg4XwhG14eJS0/edit), pronta para colar no relatório da dupla. As seções seguem a numeração do modelo.

- **Números de antes:** vieram do commit base 4e5faa3, com as mesmas ferramentas. Antes de colar, confira com metrics-before-\* (gerados pelo Enzo).
- **Números de depois:** valem só para a branch refatoracao-test-smells-matheus. Os da dupla saem de metrics-after-\*, gerados pelo Enzo depois do merge das duas branches.
- **IDs:** R01 a R20 são provisórios. Ao juntar com o Enzo, renumerem conforme a ordem dos commits na branch final.
- **Prints:** o modelo pede capturas de tela do antes e depois. Os trechos da Seção 3.3 servem de referência. O print pode ser tirado do diff de cada commit no GitHub (link em cada exemplo).

## 1. Apresentação do projeto

O [boltons](https://github.com/mahmoud/boltons) é uma biblioteca de utilitários em Python puro, sem dependências externas. Ela complementa a biblioteca padrão com estruturas e funções como caches LRU/LRI (cacheutils), dicionários multivalorados e bidirecionais (dictutils), recursão sobre estruturas aninhadas (iterutils) e arquivos temporários com transbordo para disco (ioutils). O público são desenvolvedores Python que querem esses recursos sem instalar pacotes maiores.

No commit base, o projeto tinha 30 módulos em boltons/ e 29 arquivos de teste em tests/, em geral um test\_\<módulo>.py por módulo. Os testes são rodados com pytest. A maioria é de funções no estilo pytest (`assert` simples, `pytest.raises`, `parametrize`). test\_ioutils.py e test\_strutils.py usam classes unittest.TestCase.

## 2. Estado inicial do projeto

### 2.1 Test smells detectados antes da refatoração (TEMPY)

TEMPY no commit base: 15 arquivos de teste analisados.

| Test smell | Ocorrências detectadas |
| --- | --- |
| Conditional Test Logic | 107 |
| Exception Handling | 29 |
| Unknown Test | 24 |
| Redundant Print | 16 |
| Non-Functional Statement | 14 |
| Verbose Test | 11 |
| Programming Paradigms Blend | 3 |
| Undefined Test | 2 |
| Sleepy Test | 1 |
| **Total** | **207** |

### 2.2 Resultados dos testes e cobertura antes da refatoração (Pytest)

| Métrica | Valor |
| --- | --- |
| Testes aprovados (Passed) | 525 |
| Testes com falha (Failed) | 0 |
| Erros (Errors) | 0 |
| Ignorados (Skipped) | 0 |
| Cobertura total (%) | 68% (linhas + branches, `--cov=boltons --cov-branch`) |

O guia interno da dupla registrou 67% de cobertura. Confiram o valor em metrics-before-pytest/coverage\_antes.json antes de colar: a diferença provavelmente vem do arredondamento ou de medir com ou sem branches.

### 2.3 Métricas do código de teste antes da refatoração (Radon)

| Métrica | Valor |
| --- | --- |
| SLOC total dos testes | 4600 |
| Complexidade ciclomática média das funções de teste | 3,57 (530 funções) |
| Índice de manutenibilidade médio dos arquivos de teste | 48,20 |

Valores calculados com o Radon sobre tests/ no commit base, com a mesma agregação do script (funções sem repetição e média simples do MI por arquivo).

## 3. Refatorações realizadas

### 3.1 Seleção das ocorrências

Matheus refatorou 20 ocorrências de 7 tipos. O CATSPY não dá um nível numérico de severidade, mas indica que atributos de qualidade cada smell prejudica. Usamos isso como critério: primeiro os smells que afetam a **confiabilidade** (o teste pode passar sem verificar o que deveria) e os que afetam mais atributos.

| Tipo | Atributos afetados (CATSPY) | Qtd. | Por que entrou |
| --- | --- | --- | --- |
| Conditional Test Logic | Legibilidade, Manutenibilidade, Confiabilidade | 3 | O tipo mais frequente (107). test\_lri tinha CC 12 e um `if` que deixava a fase inicial do cache sem verificação |
| Unknown Test | Manutenibilidade, Confiabilidade | 4 | Testes que passam sem nenhuma asserção explícita |
| Undefined Test | Manutenibilidade, Confiabilidade | 1 | Asserções que só rodavam como efeito colateral de outros testes |
| Non-Functional Statement | Manutenibilidade, Confiabilidade | 5 | `except: pass` usado para esperar exceções, com `assert False` dentro do `try` |
| Exception Handling | Legibilidade, Confiabilidade | 3 | Tratamento manual que escondia uma regressão (R11) |
| Verbose Test | Legibilidade, Manutenibilidade, Eficiência | 1 | test\_one\_to\_one, com 58 linhas e 8 responsabilidades |
| Redundant Print | Eficiência | 3 | O de menor severidade, mas os prints eram verificações de `__repr__` que nunca falhavam |

A divisão foi por arquivo, para os commits das duas pessoas não conflitarem. Matheus ficou com test\_cacheutils, test\_dictutils, test\_iterutils, test\_listutils, test\_ioutils e test\_debugutils\_trace.

### 3.2 Lista das refatorações

Ordem dos commits na branch [refatoracao-test-smells-matheus](https://github.com/harielfr/boltons/commits/refatoracao-test-smells-matheus). Em todas: testes passaram = sim, removido pelo TEMPY = sim, novo smell = não.

| ID | Test smell | Arquivo::teste | Commit | Testes passaram? | Removido pelo TEMPY? | Introduziu novo smell? |
| --- | --- | --- | --- | --- | --- | --- |
| R01 | Redundant Print | tests/test\_cacheutils.py::test\_callable\_cached\_dec | dac7ea0 | sim | sim | não |
| R02 | Redundant Print | tests/test\_cacheutils.py::test\_cachedmethod | 19fbfde | sim | sim | não |
| R03 | Redundant Print | tests/test\_cacheutils.py::test\_cachedmethod | b9c420e | sim | sim | não |
| R04 | Non-Functional Statement | tests/test\_cacheutils.py::test\_lru\_basic | c300b43 | sim | sim | não |
| R05 | Non-Functional Statement | tests/test\_cacheutils.py::test\_lru\_basic | da77277 | sim | sim | não |
| R06 | Non-Functional Statement | tests/test\_debugutils\_trace.py::test\_trace\_exc | 891537d | sim | sim | não |
| R07 | Non-Functional Statement | tests/test\_dictutils.py::test\_pop | a65a643 | sim | sim | não |
| R08 | Non-Functional Statement | tests/test\_dictutils.py::test\_pop\_all | f2fb31d | sim | sim | não |
| R09 | Undefined Test | tests/test\_listutils.py::\_multi\_sublist\_bl | e1c3777 | sim | sim | não |
| R10 | Exception Handling | tests/test\_ioutils.py::test\_rollover\_custom\_directory | cfa381d | sim | sim | não |
| R11 | Exception Handling | tests/test\_ioutils.py::test\_buffered\_writer\_flush | 8bf67d1 | sim | sim | não |
| R12 | Exception Handling | tests/test\_ioutils.py::test\_buffered\_writer\_flush | 3ebbda6 | sim | sim | não |
| R13 | Unknown Test | tests/test\_cacheutils.py::test\_lri\_cache\_eviction | f0e33cb | sim | sim | não |
| R14 | Unknown Test | tests/test\_iterutils.py::test\_unremappable | 7cc668c | sim | sim | não |
| R15 | Unknown Test | tests/test\_iterutils.py::test\_noncallables | 4306458 | sim | sim | não |
| R16 | Unknown Test | tests/test\_dictutils.py::test\_frozendict\_ior | 7ea9709 | sim | sim | não |
| R17 | Verbose Test | tests/test\_dictutils.py::test\_one\_to\_one | c624b10 | sim | sim | não |
| R18 | Conditional Test Logic | tests/test\_cacheutils.py::test\_lri (if) | 7ced183 | sim | sim | não |
| R19 | Conditional Test Logic | tests/test\_cacheutils.py::test\_lri (for) | 175e86f | sim | sim | não |
| R20 | Conditional Test Logic | tests/test\_dictutils.py::test\_clear | 2984e57 | sim | sim | não |

R04, R05, R07 e R08 também removeram o Exception Handling do mesmo bloco, e R18 removeu um segundo `for` de test\_lri. Esses 5 smells extras não estão contados nas 20.

### 3.3 Exemplos detalhados

Um exemplo por tipo de smell. O Radon soma +1 na complexidade ciclomática para cada `assert`. Por isso, refatorações que acrescentam asserções aumentam o CC mesmo sem criar desvios de fluxo. Cada título leva ao diff do commit, que pode ser usado como print.

#### [R01](https://github.com/harielfr/boltons/commit/dac7ea0) – Redundant Print (tests/test\_cacheutils.py::test\_callable\_cached\_dec)

```python
# Antes
    print(repr(func))

# Depois
    assert repr(func) == f'CachedFunction(func={inner_func!r})'
```

**Por que era um test smell:** o print exercitava o `__repr__`, mas nenhum resultado era verificado. O teste passaria com qualquer saída e ainda poluía o console.

**Estratégia do CATSPY aplicada:** trocar o print por uma asserção com a informação relevante. Ponto avaliado: o CATSPY também lista Sensitive Equality, mas aqui o `__repr__` é a própria unidade testada, então comparar a string é o correto.

| Métrica | Antes | Depois |
| --- | --- | --- |
| CC da função | 6 | 7 |
| SLOC de test\_cacheutils.py | 348 | 348 |

#### [R04](https://github.com/harielfr/boltons/commit/c300b43) – Non-Functional Statement (tests/test\_cacheutils.py::test\_lru\_basic)

```python
# Antes
    try:
        lru.pop('bye')
    except KeyError:
        pass
    else:
        assert False

# Depois
    with pytest.raises(KeyError):
        lru.pop('bye')
```

**Por que era um test smell:** o `pass` não faz nada. A verificação real estava espalhada entre `except` e `else: assert False`, uma lógica de controle manual difícil de ler.

**Estratégia do CATSPY aplicada:** remover a instrução sem finalidade, junto com a estratégia de Exception Handling (assertRaises → `pytest.raises`). Adaptação: o `pass` não podia ser apagado sozinho, então o bloco inteiro foi substituído.

| Métrica | Antes | Depois |
| --- | --- | --- |
| CC da função | 26 | 23 |
| SLOC de test\_cacheutils.py | 350 | 346 |

#### [R09](https://github.com/harielfr/boltons/commit/e1c3777) – Undefined Test (tests/test\_listutils.py::\_multi\_sublist\_bl)

```python
# Antes
def _multi_sublist_bl(n=30000):
    bl = BarrelList(range(n))
    bl.pop(0)
    bl.insert(0, 0)
    assert len(bl.lists) > 1
    assert list(bl) == list(range(n))
    return bl

# Depois
def _multi_sublist_bl(n=30000):
    bl = BarrelList(range(n))
    bl.pop(0)
    bl.insert(0, 0)
    return bl


def test_multi_sublist_bl_splits_and_keeps_order():
    n = 30000
    bl = _multi_sublist_bl(n)
    assert len(bl.lists) > 1
    assert list(bl) == list(range(n))
```

**Por que era um test smell:** a função tinha asserções, mas o pytest não a reconhecia como teste. As verificações só rodavam de forma implícita, dentro de outros 4 testes.

**Estratégia do CATSPY aplicada:** renomear com o prefixo `test`. Adaptação: renomear a fábrica a faria ser coletada como teste que retorna valor (Return in Test). Por isso as asserções foram movidas para um teste novo, e a fábrica ficou só com a construção.

| Métrica | Antes | Depois |
| --- | --- | --- |
| CC de \_multi\_sublist\_bl | 3 | 1 |
| CC do teste novo | – | 3 |
| SLOC de test\_listutils.py | 195 | 198 |

#### [R11](https://github.com/harielfr/boltons/commit/8bf67d1) – Exception Handling (tests/test\_ioutils.py::test\_buffered\_writer\_flush)

```python
# Antes
                finally:
                    try:
                        writer.close()
                    except BlockingIOError:
                        stream.close()

# Depois
                finally:
                    writer.close()
```

**Por que era um test smell:** o `except` engolia o BlockingIOError que o `BufferedWriter` lança quando o stream bruto retorna None em `write`. Esse era justamente o bug corrigido no PR #513. Se ele voltasse, o teste não acusaria.

**Estratégia do CATSPY aplicada:** deixar o framework tratar a exceção em vez de capturar manualmente. Aqui isso significa não capturar: uma exceção inesperada deve derrubar o teste. No commit seguinte (R12), o `try/finally` externo virou `with io.BufferedWriter(stream) as writer:`.

| Métrica | Antes | Depois |
| --- | --- | --- |
| CC da função | 4 | 3 |
| SLOC de test\_ioutils.py | 406 | 403 |

#### [R13](https://github.com/harielfr/boltons/commit/f0e33cb) – Unknown Test (tests/test\_cacheutils.py::test\_lri\_cache\_eviction)

```python
# Antes
    test_cache["key3"] = "value3"

# Depois
    test_cache["key3"] = "value3"
    assert dict(test_cache) == {"key2": "value2", "key3": "value3"}, \
        "LRI(2) should evict key1 and keep only the two newest keys"
```

**Por que era um test smell:** o teste de regressão só garantia que nenhuma exceção era lançada. Uma política de remoção errada (descartar key2 em vez de key1) passaria.

**Estratégia do CATSPY aplicada:** adicionar a asserção que valida o estado esperado, com mensagem explicativa, como no exemplo do catálogo.

| Métrica | Antes | Depois |
| --- | --- | --- |
| CC da função | 1 | 2 |
| SLOC de test\_cacheutils.py | 342 | 344 |

#### [R17](https://github.com/harielfr/boltons/commit/c624b10) – Verbose Test (tests/test\_dictutils.py::test\_one\_to\_one)

```python
# Antes: 58 linhas, um único teste
def test_one_to_one():
    e = OneToOne({1:2})
    def ck(val, inv):
        assert (e, e.inv) == (val, inv)
    ck({1:2}, {2:1})
    e[2] = 3
    ...  # setitem, delitem, copy, pop, setdefault, update, sobreposição, unique

# Depois: 8 testes focados, por exemplo
def test_one_to_one_pop():
    e = OneToOne({1:2})
    e.pop(1)
    assert (e, e.inv) == ({}, {})
    e[1] = 2
    e.inv.pop(2)
    assert (e, e.inv) == ({}, {})
    e[1] = 2
    e.popitem()
    assert (e, e.inv) == ({}, {})
```

**Por que era um test smell:** um teste com 8 responsabilidades, em que cada passo dependia do estado deixado pelo anterior. Uma falha no começo escondia todas as verificações seguintes.

**Estratégia do CATSPY aplicada:** dividir em métodos menores, um por responsabilidade, cada um com seu próprio estado inicial. Adaptação: a primeira versão usava um helper de asserção, e o TEMPY acusou 7 smells novos (Undefined e Unknown Test). A versão final usa asserções explícitas.

| Métrica | Antes | Depois |
| --- | --- | --- |
| CC da função | 6 (1 função) | 2 a 7 (8 funções; máx. 7 em setitem) |
| SLOC de test\_dictutils.py | 452 | 459 |

#### [R18](https://github.com/harielfr/boltons/commit/7ced183) e [R19](https://github.com/harielfr/boltons/commit/175e86f) – Conditional Test Logic (tests/test\_cacheutils.py::test\_lri)

```python
# Antes
    for idx, char in enumerate(string.ascii_letters):
        x = bc[char]
        assert x == char.upper()
        least_recent_insert_index = idx - cache_size
        if least_recent_insert_index >= 0:
            assert len(bc) == cache_size
            for char in string.ascii_letters[least_recent_insert_index+1:idx]:
                assert char in bc

# Depois (R18 + R19)
@pytest.mark.parametrize("lookup_count", range(1, len(string.ascii_letters) + 1))
def test_lri_on_miss_keeps_most_recent_inserts(lookup_count):
    cache_size = 10
    bc = LRI(cache_size, on_miss=lambda k: k.upper())
    looked_up = string.ascii_letters[:lookup_count]
    assert [bc[char] for char in looked_up] == list(looked_up.upper())
    assert set(bc) == set(looked_up[-cache_size:])
```

**Por que era um test smell:** o `if` deixava os 10 primeiros passos sem verificação de conteúdo, e os laços aninhados escondiam qual passo falhou. A função tinha CC 12, empatada em segundo lugar no arquivo, atrás de test\_lru\_basic (26).

**Estratégia do CATSPY aplicada:** extrair cada caso para um teste separado. R18 substituiu o `if` e o `for` interno por uma asserção única, válida em todos os passos. R19 transformou o `for` externo em `parametrize` (52 casos). O teste ficou mais rigoroso: compara o conjunto exato, então também pega chaves sobrando.

| Métrica | Antes | Depois |
| --- | --- | --- |
| CC de test\_lri | 12 | 9 (R18), depois 7 (R19) |
| CC do teste parametrizado | – | 3 |
| SLOC de test\_cacheutils.py | 344 | 341 (R18), depois 344 (R19) |

## 4. Estado final e comparação

As 20 refatorações de Matheus tiraram 25 smells do projeto, mantiveram todos os testes passando e não mudaram a cobertura. A tabela mede só a branch refatoracao-test-smells-matheus. Na versão final, use metrics-after-\*, que soma as duas pessoas.

| Métrica | Antes | Depois (só Matheus) | Variação |
| --- | --- | --- | --- |
| Total de test smells (TEMPY) | 207 | 182 | −25 |
| Testes aprovados | 525 | 589 | +64 |
| Testes com falha | 0 | 0 | 0 |
| Cobertura total (%) | 68 | 68 | 0 p.p. |
| SLOC dos testes | 4600 | 4595 | −5 |
| Complexidade ciclomática média das funções de teste | 3,57 | 3,55 | −0,02 |
| Índice de manutenibilidade médio dos arquivos de teste | 48,20 | 48,11 | −0,09 |

| Test smell | Antes | Depois (só Matheus) | Variação |
| --- | --- | --- | --- |
| Conditional Test Logic | 107 | 103 | −4 |
| Exception Handling | 29 | 22 | −7 |
| Unknown Test | 24 | 20 | −4 |
| Redundant Print | 16 | 13 | −3 |
| Non-Functional Statement | 14 | 9 | −5 |
| Verbose Test | 11 | 10 | −1 |
| Programming Paradigms Blend | 3 | 3 | 0 |
| Undefined Test | 2 | 1 | −1 |
| Sleepy Test | 1 | 1 | 0 |
| **Total** | **207** | **182** | **−25** |

## 5. Análise dos resultados

**Quais métricas melhoraram e quais pioraram?** Os smells caíram 12% (207 para 182), e o CC médio das funções de teste caiu levemente (3,57 para 3,55). As maiores quedas foram em test\_lri (12 para 7) e test\_lru\_basic (26 para 20). O MI médio caiu 0,09 ponto, puxado por test\_dictutils.py (6,28 para 3,42): dividir test\_one\_to\_one em 8 funções aumentou o volume de Halstead do arquivo, e o MI é calculado por arquivo, não por função. Em várias refatorações o CC da função subiu 1 (R01, R13, R14, R16) porque o Radon conta cada `assert` como ponto de decisão. Nesses casos o aumento reflete mais verificação, não mais complexidade.

**Alguma refatoração introduziu novos test smells? Quais?** Não, segundo o TEMPY, conferido após cada commit. A primeira versão de R17 introduzia 7 (1 Undefined Test e 6 Unknown Test, por causa de um helper de asserção) e foi descartada antes do commit. Fora do que o TEMPY detecta: R01 a R03 comparam strings de `repr`, o que lembra Sensitive Equality. Ali, porém, o `__repr__` é o comportamento testado.

**O resultado dos testes ou a cobertura mudou?** Nenhum teste passou a falhar. O número de testes subiu de 525 para 589 por causa de parametrizações (+52 em test\_lri\_on\_miss\_keeps\_most\_recent\_inserts, +4 em test\_clear), da divisão de test\_one\_to\_one (+7) e do teste extraído de \_multi\_sublist\_bl (+1). A cobertura ficou em 68%, o esperado: as mudanças foram só nos testes e preservaram o que era exercitado.

**Quais tipos de smell foram mais fáceis e mais difíceis de refatorar?** Mais fáceis: Non-Functional Statement e Exception Handling no padrão `try/except/pass`, que têm substituição direta por `pytest.raises`. Mais difíceis: Conditional Test Logic em test\_lri (laços aninhados com estado acumulado) e Verbose Test em test\_one\_to\_one (cada passo dependia do anterior). Também exigiram cuidado os casos em que a receita do catálogo não servia como estava: Undefined Test em uma fábrica, e Exception Handling em `try/finally` de limpeza.

## 6. Uso do CATSPY

**Como você usou o CATSPY durante a atividade?** Antes de cada refatoração, abri a página do smell e comparei o trecho do boltons com o exemplo "com problema". Depois usei o exemplo refatorado como ponto de partida. Usei também a lista geral, com os atributos de qualidade afetados, para priorizar os tipos (Seção 3.1). E consultei smells vizinhos para checar se a solução criaria outro problema: Sensitive Equality em R01–R03, Return in Test em R09.

**Quais partes do catálogo mais ajudaram?** O exemplo refatorado, principalmente em Exception Handling (assertRaises → `pytest.raises`), Unknown Test (asserção com mensagem) e Verbose Test (dividir por responsabilidade). A seção de efeitos negativos ajudou a explicar no diário por que cada trecho era um problema. Os exemplos "em outros contextos" ajudaram pouco num projeto de biblioteca.

**Houve casos em que a estratégia do catálogo não se aplicava diretamente ao projeto? Como você resolveu?** Sim, em quatro situações:

- **Undefined Test (R09):** renomear a fábrica a faria virar um teste que retorna valor. Separei construção e verificação.
- **Non-Functional Statement (R06):** o `pass` era o corpo de uma classe de exceção, e não um método vazio. Troquei por uma docstring.
- **Exception Handling (R10, R12):** os `try` eram de limpeza de recurso (`try/finally`), e não de espera de exceção. Usei gerenciadores de contexto (`TemporaryDirectory`, `with BufferedWriter`).
- **Unknown Test (R14, R15):** os testes usavam `pytest.raises`, que o TEMPY não conta como asserção. Acrescentei asserções sobre a mensagem da exceção, o que também torna o teste mais preciso.

**O que faltou ou poderia ser melhorado no catálogo?** Exemplos em estilo pytest (funções com `assert`, `pytest.raises`, `parametrize`): quase todos estão em unittest, e a maioria dos testes do boltons é pytest. Também faltam: o caso de Exception Handling com `try/finally` de limpeza, um alerta de que renomear helpers com parâmetros ou retorno quebra o pytest (Undefined Test), um nível explícito de severidade para priorizar, e uma nota de que a definição de Verbose Test (mais de 30 linhas) difere do limite do TEMPY (45 linhas).

## 7. Divisão do trabalho

| Integrante | Refatorações realizadas (IDs da Seção 3.2) | Quantidade |
| --- | --- | --- |
| Enzo Hariel | \[preencher: IDs do Enzo\] | 20 |
| Matheus dos Santos | R01 a R20 da branch refatoracao-test-smells-matheus (renumerar após o merge) | 20 |

A divisão foi por arquivo de teste, para os commits não conflitarem no merge. Cada integrante ficou com 20 ocorrências de dificuldades variadas.

## 8. Dificuldades e lições aprendidas

**Quais foram as principais dificuldades?**

- **Ferramentas:** o TEMPY detecta Conditional Test Logic por texto (linhas que começam com `if`, `for` ou `while`) e não conta `pytest.raises` nem asserções dentro de helpers. Foi preciso conferir cada arquivo após a mudança para não criar smells novos (caso do R17).
- **Entendimento do código:** em R11 foi preciso ler `SpooledBytesIO.write` e o histórico do PR #513 para concluir que o `except BlockingIOError` escondia uma regressão.
- **Commits atômicos:** em trechos aninhados (R11/R12, R18/R19), refatorei primeiro o bloco interno para manter uma ocorrência por commit.
- **Ambiente:** no Windows, o arquivo temporário precisa ser fechado antes de apagar o diretório (R10). O Git avisou sobre conversão LF/CRLF, sem efeito no diff.
- **Upstream:** o mahmoud/boltons avançou 28 commits além do commit base. Testei o rebase da branch: sem conflito, 684 testes aprovados.

**O que você aprendeu com a atividade?** Que um teste pode passar sem verificar nada (Unknown Test, `except: pass`, prints) e que esses são os smells mais perigosos. Que um detector é um ponto de partida, não um juiz: várias ocorrências pediam um julgamento diferente da receita padrão. E que parametrizar e dividir testes aumenta o número de testes, mas deixa cada falha muito mais fácil de localizar.
