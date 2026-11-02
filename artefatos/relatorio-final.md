# Relatório Final

**Disciplina:** Qualidade de Software (UFC Quixadá, Profa. Carla Ilane Moreira Bezerra)\
**Equipe:** Enzo Hariel - 566785 e Matheus dos Santos - 563640\
**Projeto:** boltons — https://github.com/mahmoud/boltons\
**Fork:** https://github.com/harielfr/boltons\
**Pull Request:** https://github.com/mahmoud/boltons/pull/710\
**Commit SHA:** `4e5faa3d7e4008d89e0d8bf1ea87b6d9a061a16d` (commit base)

## 1. Apresentação do projeto

O [boltons](https://github.com/mahmoud/boltons) é uma biblioteca de utilitários em Python puro, sem dependências externas. Ela complementa a biblioteca padrão com estruturas e funções como caches LRU/LRI (`cacheutils`), dicionários multivalorados e bidirecionais (`dictutils`), recursão sobre estruturas aninhadas (`iterutils`), sockets com buffer (`socketutils`) e arquivos temporários com transbordo para disco (`ioutils`). O público são desenvolvedores Python que querem esses recursos sem instalar pacotes maiores.

No commit base (`4e5faa3`), o projeto tinha 29 módulos em `boltons/` (fora o `__init__.py`) e 29 arquivos de teste em `tests/`, em geral um `test_<módulo>.py` por módulo. Os testes rodam com pytest. A maioria é de funções no estilo pytest (`assert` simples, `pytest.raises`, `parametrize`); `test_ioutils.py` e `test_strutils.py` usam classes `unittest.TestCase`. O TEMPY analisou 15 desses arquivos.

## 2. Estado inicial do projeto

### 2.1 Test smells detectados antes da refatoração (TEMPY)

Fonte: `metrics-before-tempy/resumo.json` (15 arquivos de teste analisados).

| Test smell                  | Ocorrências detectadas |
| --------------------------- | ---------------------- |
| Conditional Test Logic      | 107                    |
| Exception Handling          | 29                     |
| Unknown Test                | 24                     |
| Redundant Print             | 16                     |
| Non-Functional Statement    | 14                     |
| Verbose Test                | 11                     |
| Programming Paradigms Blend | 3                      |
| Undefined Test              | 2                      |
| Sleepy Test                 | 1                      |
| **Total**                   | **207**                |

### 2.2 Resultados dos testes e cobertura antes da refatoração (Pytest)

Fonte: `metrics-before-pytest/` (`pytest-cov` com `--cov-branch`).

| Métrica                   | Valor                                           |
| ------------------------- | ----------------------------------------------- |
| Testes aprovados (Passed) | 525 (mais 12 subtests)                          |
| Testes com falha (Failed) | 0                                               |
| Erros (Errors)            | 0                                               |
| Ignorados (Skipped)       | 0                                               |
| Cobertura total (%)       | 67,37% (linhas e branches; 5181 de 7326 linhas) |

### 2.3 Métricas do código de teste antes da refatoração (Radon)

Fonte: linhas `origem = teste` de `metrics-before-radon/` (linha TOTAL).

| Métrica                                                | Valor              |
| ------------------------------------------------------ | ------------------ |
| SLOC total dos testes (linha TOTAL com origem = teste) | 4600               |
| Complexidade ciclomática média das funções de teste    | 3,57 (530 funções) |
| Índice de manutenibilidade médio dos arquivos de teste | 48,20              |

## 3. Refatorações realizadas

### 3.1 Seleção das ocorrências

Foram **41 refatorações de 8 tipos**, cada uma com um commit: 20 de Enzo mais 1 extra dele (R21), e 20 de Matheus. O CATSPY não dá um nível numérico de severidade, mas indica quais atributos de qualidade cada smell prejudica. Usamos isso como critério: primeiro os smells que afetam a **confiabilidade** (o teste pode passar sem verificar o que deveria) e os que afetam mais atributos.

| Tipo                     | Atributos afetados (CATSPY)                    | Qtd. | Por que entrou                                                                                                          |
| ------------------------ | ---------------------------------------------- | ---- | ----------------------------------------------------------------------------------------------------------------------- |
| Conditional Test Logic   | Legibilidade, Manutenibilidade, Confiabilidade | 6    | É o tipo mais frequente (107 de 207). `test_lri` tinha CC 12, e `test_multibyte_delim` repetia o corpo inteiro num laço |
| Unknown Test             | Manutenibilidade, Confiabilidade               | 8    | Testes que passam sem nenhuma asserção explícita (o TEMPY não conta `pytest.raises`)                                    |
| Undefined Test           | Manutenibilidade, Confiabilidade               | 2    | Asserções que o pytest nunca coleta (`_test_bad_utf8`) ou só roda como efeito colateral (`_multi_sublist_bl`)           |
| Non-Functional Statement | Manutenibilidade, Confiabilidade               | 10   | `except: pass` usado para esperar exceções, com `assert False` no `else`                                                |
| Exception Handling       | Legibilidade, Confiabilidade                   | 5    | Tratamento manual de exceção, que em um caso escondia uma regressão                                                     |
| Sleepy Test              | Eficiência, Confiabilidade                     | 1    | `time.sleep` como cenário do teste de timeout                                                                           |
| Verbose Test             | Legibilidade, Manutenibilidade, Eficiência     | 2    | `test_wraps_injected` (49 linhas, 5 cenários) e `test_one_to_one` (58 linhas, 8 responsabilidades)                      |
| Redundant Print          | Eficiência                                     | 7    | O de menor severidade; os prints de Matheus eram verificações de `__repr__` que nunca falhavam                          |

A divisão foi por arquivo de teste, para os commits das duas pessoas não conflitarem no merge. Enzo ficou com `test_socketutils`, `test_urlutils`, `test_namedutils`, `test_mathutils`, `test_funcutils_fb` e `test_setutils`. Matheus ficou com `test_cacheutils`, `test_dictutils`, `test_iterutils`, `test_listutils`, `test_ioutils` e `test_debugutils_trace`.

### 3.2 Lista das refatorações

Ordem: R01 a R21 são de Enzo, na ordem dos commits da branch `refatoracao-test-smells`; R22 a R41 são de Matheus, na ordem da branch `refatoracao-test-smells-matheus` (R01 a R20 no relatório individual dele). Links dos commits: `https://github.com/harielfr/boltons/commit/<hash>`. A coluna "Testes passaram?" vale para o arquivo e para a suíte completa nos checkpoints; a suíte final tem 594 passed e 1 xfailed.

| ID  | Test smell                       | Arquivo::teste                                                            | Commit  | Testes passaram? | Removido pelo TEMPY? | Introduziu novo smell?                                               |
| --- | -------------------------------- | ------------------------------------------------------------------------- | ------- | ---------------- | -------------------- | -------------------------------------------------------------------- |
| R01 | Redundant Print                  | tests/test_socketutils.py::test_socketutils_netstring                     | 7270697 | sim              | sim                  | não                                                                  |
| R02 | Redundant Print                  | tests/test_socketutils.py::test_socketutils_netstring                     | 76ef5c7 | sim              | sim                  | não                                                                  |
| R03 | Redundant Print                  | tests/test_socketutils.py::test_socketutils_netstring                     | 06d64ae | sim              | sim                  | não                                                                  |
| R04 | Redundant Print                  | tests/test_socketutils.py::test_socketutils_netstring                     | b923cec | sim              | sim                  | sim: Non-Functional Statement (`pass` no `except`), corrigido em R21 |
| R05 | Non-Functional Statement         | tests/test_socketutils.py::test_short_lines                               | e880ae3 | sim              | sim                  | não                                                                  |
| R06 | Non-Functional Statement         | tests/test_socketutils.py::test_client_disconnecting                      | 76313f8 | sim              | sim                  | não                                                                  |
| R07 | Non-Functional Statement         | tests/test_socketutils.py::test_client_disconnecting                      | 7b72f5c | sim              | sim                  | não                                                                  |
| R08 | Non-Functional Statement         | tests/test_socketutils.py::test_split_delim                               | 5ae6e58 | sim              | sim                  | não                                                                  |
| R09 | Sleepy Test                      | tests/test_socketutils.py::test_socketutils_netstring_timeout             | d8226a1 | sim              | sim                  | não                                                                  |
| R10 | Undefined Test                   | tests/test_urlutils.py::\_test_bad_utf8                                   | 3ca263c | sim (1 xfail)    | sim                  | não                                                                  |
| R11 | Exception Handling               | tests/test_socketutils.py::test_basic_nonblocking                         | 7c7fbc8 | sim              | sim                  | não                                                                  |
| R12 | Exception Handling               | tests/test_socketutils.py::test_basic_nonblocking                         | 56c55d6 | sim              | sim                  | não                                                                  |
| R13 | Unknown Test                     | tests/test_namedutils.py::test_empty_field_name_raises_value_error        | 341557d | sim              | sim                  | não                                                                  |
| R14 | Unknown Test                     | tests/test_namedutils.py::test_empty_type_name_raises_value_error         | 0107779 | sim              | sim                  | não                                                                  |
| R15 | Unknown Test                     | tests/test_namedutils.py::test_invalid_character_still_raises_value_error | e18cb8d | sim              | sim                  | não                                                                  |
| R16 | Unknown Test                     | tests/test_mathutils.py::test_ceil_oor_upper                              | a98b4ab | sim              | sim                  | não                                                                  |
| R17 | Verbose Test                     | tests/test_funcutils_fb.py::test_wraps_injected                           | 3b1d48a | sim              | sim                  | não                                                                  |
| R18 | Conditional Test Logic           | tests/test_socketutils.py::test_multibyte_delim (for)                     | 9a1f762 | sim              | sim                  | não                                                                  |
| R19 | Conditional Test Logic           | tests/test_socketutils.py::test_multibyte_delim (if)                      | ab72792 | sim              | sim                  | não                                                                  |
| R20 | Conditional Test Logic           | tests/test_setutils.py::test_iset_index_method                            | 989f837 | sim              | sim                  | não                                                                  |
| R21 | Non-Functional Statement (extra) | tests/test_socketutils.py::test_socketutils_netstring                     | 6aa6b6b | sim              | sim                  | não                                                                  |
| R22 | Redundant Print                  | tests/test_cacheutils.py::test_callable_cached_dec                        | dac7ea0 | sim              | sim                  | não                                                                  |
| R23 | Redundant Print                  | tests/test_cacheutils.py::test_cachedmethod                               | 19fbfde | sim              | sim                  | não                                                                  |
| R24 | Redundant Print                  | tests/test_cacheutils.py::test_cachedmethod                               | b9c420e | sim              | sim                  | não                                                                  |
| R25 | Non-Functional Statement         | tests/test_cacheutils.py::test_lru_basic                                  | c300b43 | sim              | sim                  | não                                                                  |
| R26 | Non-Functional Statement         | tests/test_cacheutils.py::test_lru_basic                                  | da77277 | sim              | sim                  | não                                                                  |
| R27 | Non-Functional Statement         | tests/test_debugutils_trace.py::test_trace_exc                            | 891537d | sim              | sim                  | não                                                                  |
| R28 | Non-Functional Statement         | tests/test_dictutils.py::test_pop                                         | a65a643 | sim              | sim                  | não                                                                  |
| R29 | Non-Functional Statement         | tests/test_dictutils.py::test_pop_all                                     | f2fb31d | sim              | sim                  | não                                                                  |
| R30 | Undefined Test                   | tests/test_listutils.py::\_multi_sublist_bl                               | e1c3777 | sim              | sim                  | não                                                                  |
| R31 | Exception Handling               | tests/test_ioutils.py::test_rollover_custom_directory                     | cfa381d | sim              | sim                  | não                                                                  |
| R32 | Exception Handling               | tests/test_ioutils.py::test_buffered_writer_flush                         | 8bf67d1 | sim              | sim                  | não                                                                  |
| R33 | Exception Handling               | tests/test_ioutils.py::test_buffered_writer_flush                         | 3ebbda6 | sim              | sim                  | não                                                                  |
| R34 | Unknown Test                     | tests/test_cacheutils.py::test_lri_cache_eviction                         | f0e33cb | sim              | sim                  | não                                                                  |
| R35 | Unknown Test                     | tests/test_iterutils.py::test_unremappable                                | 7cc668c | sim              | sim                  | não                                                                  |
| R36 | Unknown Test                     | tests/test_iterutils.py::test_noncallables                                | 4306458 | sim              | sim                  | não                                                                  |
| R37 | Unknown Test                     | tests/test_dictutils.py::test_frozendict_ior                              | 7ea9709 | sim              | sim                  | não                                                                  |
| R38 | Verbose Test                     | tests/test_dictutils.py::test_one_to_one                                  | c624b10 | sim              | sim                  | não                                                                  |
| R39 | Conditional Test Logic           | tests/test_cacheutils.py::test_lri (if)                                   | 7ced183 | sim              | sim                  | não                                                                  |
| R40 | Conditional Test Logic           | tests/test_cacheutils.py::test_lri (for)                                  | 175e86f | sim              | sim                  | não                                                                  |
| R41 | Conditional Test Logic           | tests/test_dictutils.py::test_clear                                       | 2984e57 | sim              | sim                  | não                                                                  |

Algumas refatorações removeram smells vizinhos junto, que não entram nas 40: R05 a R08 (Enzo) e R25, R26, R28, R29 (Matheus) removeram um Exception Handling cada; R21 removeu mais um; R39 removeu um `for` interno de `test_lri`. Total: 5 de Enzo e 5 de Matheus, que somados às 40 principais dão os −50 do TEMPY. O Non-Functional Statement criado pelo R04 e removido pelo R21 se anula.

### 3.3 Exemplos detalhados

Um exemplo por tipo de smell (dois ou mais nos tipos em que as duas pessoas adaptaram a receita de formas diferentes). Em vez de prints de tela, cada título leva ao diff do commit no GitHub, e o trecho está transcrito em bloco de código. O Radon soma +1 na complexidade ciclomática para cada `assert`, então refatorações que acrescentam asserções aumentam o CC sem criar desvios de fluxo.

> Nas tabelas de métricas, a CC é a da função no estado final (commit base contra `integracao`) e o SLOC é o do arquivo inteiro nas mesmas duas pontas. Como Enzo fez vários commits em `test_socketutils.py`, o SLOC desse arquivo (313 para 291) é o efeito acumulado de todos eles, e não só do exemplo mostrado. Idem a CC das funções que tiveram mais de uma refatoração (`test_socketutils_netstring`, `test_client_disconnecting`, `test_basic_nonblocking`, `test_multibyte_delim`).

#### [R01](https://github.com/harielfr/boltons/commit/7270697) — Redundant Print (tests/test_socketutils.py::test_socketutils_netstring)

**Antes:**

```python
    """A holistic feature test of BufferedSocket via the NetstringSocket
    wrapper. Runs
    """
    print("running self tests")

    # Set up server
```

**Depois:**

```python
    """A holistic feature test of BufferedSocket via the NetstringSocket
    wrapper. Runs
    """

    # Set up server
```

**Por que era um test smell:** o print só imprimia uma string fixa e não verificava nada. O resultado do teste vem das asserções, e o print acrescenta ruído e complexidade.

**Estratégia do CATSPY aplicada:** remover o print. Não havia o que transformar em asserção, e nas R02 e R03 o print mostrava uma medida de tempo, que viraria asserção instável.

**Métricas do teste refatorado:**

| Métrica                                            | Antes | Depois |
| -------------------------------------------------- | ----- | ------ |
| Complexidade ciclomática da função (cc_por_funcao) | 16    | 15     |
| SLOC do arquivo de teste (raw_por_arquivo_e_total) | 313   | 291    |

#### [R22](https://github.com/harielfr/boltons/commit/dac7ea0) — Redundant Print (tests/test_cacheutils.py::test_callable_cached_dec)

**Antes:**

```python
    print(repr(func))
```

**Depois:**

```python
    assert repr(func) == f'CachedFunction(func={inner_func!r})'
```

**Por que era um test smell:** o print exercitava o `__repr__`, mas nenhum resultado era verificado. O teste passaria com qualquer saída.

**Estratégia do CATSPY aplicada:** trocar o print por uma asserção com a informação relevante (exemplo refatorado do catálogo). Foi avaliado se isso criaria Sensitive Equality; como o `__repr__` é a própria unidade testada, comparar a string é a verificação correta.

| Métrica                                            | Antes | Depois |
| -------------------------------------------------- | ----- | ------ |
| Complexidade ciclomática da função (cc_por_funcao) | 6     | 7      |
| SLOC do arquivo de teste (raw_por_arquivo_e_total) | 348   | 344    |

#### [R07](https://github.com/harielfr/boltons/commit/7b72f5c) — Non-Functional Statement (tests/test_socketutils.py::test_client_disconnecting)

**Antes:**

```python
    try:
        by.send(b'123')
    except OSError:
        pass
    else:
        assert False, 'expected socket.error on closed send'
```

**Depois:**

```python
    with pytest.raises(OSError):
        by.send(b'123')
```

**Por que era um test smell:** o `pass` não faz nada, e a verificação real ficava espalhada entre o `except` e o `else: assert False`.

**Estratégia do CATSPY aplicada:** remover a instrução sem finalidade, combinada com a estratégia de Exception Handling (`assertRaises`, que no pytest é `pytest.raises`). O `pass` não podia ser apagado sozinho, então o bloco inteiro foi substituído. No R08 (`5ae6e58`) o `except Timeout: pass` só _tolerava_ a exceção, sem exigi-la; trocar por `pytest.raises` deixaria o teste mais rígido, e por isso usamos `contextlib.suppress(Timeout)`.

**Métricas do teste refatorado:**

| Métrica                                            | Antes | Depois |
| -------------------------------------------------- | ----- | ------ |
| Complexidade ciclomática da função (cc_por_funcao) | 19    | 13     |
| SLOC do arquivo de teste (raw_por_arquivo_e_total) | 313   | 291    |

#### [R25](https://github.com/harielfr/boltons/commit/c300b43) — Non-Functional Statement (tests/test_cacheutils.py::test_lru_basic)

**Antes:**

```python
    try:
        lru.pop('bye')
    except KeyError:
        pass
    else:
        assert False
```

**Depois:**

```python
    with pytest.raises(KeyError):
        lru.pop('bye')
```

**Por que era um test smell:** o `pass` não faz nada, e a lógica de "esperar uma exceção" estava escrita à mão, difícil de ler.

**Estratégia do CATSPY aplicada:** remover a instrução sem finalidade, junto com a estratégia de Exception Handling. No R27 (`891537d`) o `pass` era o corpo de uma classe de exceção local, onde algum comando é obrigatório; a receita foi adaptada trocando o `pass` por uma docstring.

| Métrica                                            | Antes | Depois |
| -------------------------------------------------- | ----- | ------ |
| Complexidade ciclomática da função (cc_por_funcao) | 26    | 20     |
| SLOC do arquivo de teste (raw_por_arquivo_e_total) | 348   | 344    |

#### [R09](https://github.com/harielfr/boltons/commit/d8226a1) — Sleepy Test (tests/test_socketutils.py::test_socketutils_netstring_timeout)

**Antes:**

```python
    client = client_connect()
    time.sleep(0.02)
    client.write_ns(b'ping')
    assert client.read_ns() == b'pong'
```

**Depois:**

```python
    client = client_connect()
    with pytest.raises(Timeout):
        client.read_ns(timeout=0.02)
    client.write_ns(b'ping')
    assert client.read_ns() == b'pong'
```

**Por que era um test smell:** o teste dependia de uma pausa de tempo fixo, que o deixa lento e pouco confiável, e nada verificava que a espera de fato ocorreu.

**Estratégia do CATSPY aplicada:** o catálogo sugere sincronizar pelo estado ou pela conclusão da tarefa, em vez de tempo fixo, ou apenas apagar o `sleep`. Foi preciso adaptar: aqui o silêncio do cliente é o cenário exercitado (o servidor lê com `read_ns(1)` e timeout padrão de 0,01 s), e sincronizar por evento exigiria mexer no servidor auxiliar. O silêncio passou a vir do próprio mecanismo de timeout, e o `pytest.raises` exige que a leitura expire. Um teste de mutação (trocar `read_ns(1)` por `read_ns()` no servidor) confirmou que o teste continua falhando quando deveria.

**Métricas do teste refatorado:**

| Métrica                                            | Antes | Depois |
| -------------------------------------------------- | ----- | ------ |
| Complexidade ciclomática da função (cc_por_funcao) | 2     | 2      |
| SLOC do arquivo de teste (raw_por_arquivo_e_total) | 313   | 291    |

#### [R10](https://github.com/harielfr/boltons/commit/3ca263c) — Undefined Test (tests/test_urlutils.py::\_test_bad_utf8)

**Antes:**

```python
def _test_bad_utf8():  # not part of the API
    bad_bin_url = 'http://xn--9ca.com/%00%FF/%C3%A9'
    url = URL(bad_bin_url)
```

**Depois:**

```python
@pytest.mark.xfail(strict=True, reason="to_text() decodifica %00/%FF em vez de mantê-los codificados")
def test_bad_utf8():  # not part of the API
    bad_bin_url = 'http://xn--9ca.com/%00%FF/%C3%A9'
    url = URL(bad_bin_url)
```

**Por que era um test smell:** a função tinha asserção, mas o nome começava com `_test`, então o pytest não a coletava e ela nunca rodava, dando a falsa impressão de que o comportamento estava coberto.

**Estratégia do CATSPY aplicada:** renomear com o prefixo `test`. Adaptação: ao rodar, a função falha (`to_text()` devolve `'http://é.com/\x00�/é'` em vez de `'http://é.com/%00%FF/é'`), então o renomeio simples poria uma falha nova na suíte. Como não era permitido alterar `boltons/` nem o valor esperado, o teste foi marcado com `xfail(strict=True)`, que documenta a falha e acusa se o código for corrigido. Isso revelou um comportamento do código-fonte que o teste escondia.

**Métricas do teste refatorado:**

| Métrica                                            | Antes | Depois |
| -------------------------------------------------- | ----- | ------ |
| Complexidade ciclomática da função (cc_por_funcao) | 2     | 2      |
| SLOC do arquivo de teste (raw_por_arquivo_e_total) | 357   | 358    |

#### [R30](https://github.com/harielfr/boltons/commit/e1c3777) — Undefined Test (tests/test_listutils.py::\_multi_sublist_bl)

**Antes:**

```python
def _multi_sublist_bl(n=30000):
    bl = BarrelList(range(n))
    bl.pop(0)
    bl.insert(0, 0)
    assert len(bl.lists) > 1
    assert list(bl) == list(range(n))
    return bl
```

**Depois:**

```python
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

**Por que era um test smell:** a função tinha asserções, mas o pytest não a reconhecia como teste; as verificações só rodavam de forma implícita, dentro de outros 4 testes.

**Estratégia do CATSPY aplicada:** renomear ou remover. Adaptação: renomear a fábrica a faria ser coletada como teste que retorna valor (Return in Test). Por isso as asserções foram movidas para um teste novo, e a fábrica ficou só com a construção.

| Métrica                                            | Antes | Depois            |
| -------------------------------------------------- | ----- | ----------------- |
| Complexidade ciclomática de `_multi_sublist_bl`    | 3     | 1 (teste novo: 3) |
| SLOC do arquivo de teste (raw_por_arquivo_e_total) | 195   | 198               |

#### [R11](https://github.com/harielfr/boltons/commit/7c7fbc8) — Exception Handling (tests/test_socketutils.py::test_basic_nonblocking)

**Antes:**

```python
    try:
        bs.recv_until(delim, timeout=0)
    except OSError as se:
        assert se.errno == errno.EWOULDBLOCK
    y.sendall(delim)  # sending an empty message, effectively
```

**Depois:**

```python
    with pytest.raises(OSError) as excinfo:
        bs.recv_until(delim, timeout=0)
    assert excinfo.value.errno == errno.EWOULDBLOCK
    y.sendall(delim)  # sending an empty message, effectively
```

**Por que era um test smell:** sem um `else: fail`, o teste passaria mesmo que nenhuma exceção ocorresse; o resultado dependia de uma exceção capturada à mão.

**Estratégia do CATSPY aplicada:** `assertRaises`, que no pytest é `pytest.raises`. Adaptação: o `try` original só tolerava a exceção, então o `pytest.raises` endurece o teste (passa a exigi-la). Para aprovar isso, confirmamos que a exceção é determinística (socketpair vazio e `timeout=0`): 30 execuções do teste isolado no R11 e 200 execuções do cenário no R12 (`56c55d6`), mais 1000 repetições na validação.

**Métricas do teste refatorado:**

| Métrica                                            | Antes | Depois |
| -------------------------------------------------- | ----- | ------ |
| Complexidade ciclomática da função (cc_por_funcao) | 10    | 8      |
| SLOC do arquivo de teste (raw_por_arquivo_e_total) | 313   | 291    |

#### [R32](https://github.com/harielfr/boltons/commit/8bf67d1) — Exception Handling (tests/test_ioutils.py::test_buffered_writer_flush)

**Antes:**

```python
                finally:
                    try:
                        writer.close()
                    except BlockingIOError:
                        stream.close()
```

**Depois:**

```python
                finally:
                    writer.close()
```

**Por que era um test smell:** o `except` engolia o `BlockingIOError` que o `BufferedWriter` lança quando o stream bruto retorna `None` em `write`, que era justamente o bug corrigido no PR #513 do upstream. Se ele voltasse, o teste não acusaria.

**Estratégia do CATSPY aplicada:** deixar o framework tratar a exceção em vez de capturar manualmente, ou seja, não capturar. No R33 (`3ebbda6`) o `try/finally` externo virou `with io.BufferedWriter(stream) as writer:`, e no R31 (`cfa381d`) um `try/finally` de limpeza virou `with TemporaryDirectory()`. A receita do catálogo (`assertRaises`) não se aplicava a esses `try` de limpeza de recurso.

| Métrica                                            | Antes | Depois |
| -------------------------------------------------- | ----- | ------ |
| Complexidade ciclomática da função (cc_por_funcao) | 4     | 3      |
| SLOC do arquivo de teste (raw_por_arquivo_e_total) | 409   | 400    |

#### [R16](https://github.com/harielfr/boltons/commit/a98b4ab) — Unknown Test (tests/test_mathutils.py::test_ceil_oor_upper)

**Antes:**

```python
def test_ceil_oor_upper():
    with raises(ValueError):
        ceil(OUT_OF_RANGE_UPPER, OPTIONS)
```

**Depois:**

```python
def test_ceil_oor_upper():
    with raises(ValueError) as exc_info:
        ceil(OUT_OF_RANGE_UPPER, OPTIONS)
    assert 'no ceil options greater than or equal to' in str(exc_info.value)
```

**Por que era um test smell:** o teste verificava a exceção com `raises`, mas o TEMPY só conta linhas com `assert`, e qualquer `ValueError` fazia o teste passar, inclusive um vindo de outro ponto.

**Estratégia do CATSPY aplicada:** terminar o teste com uma asserção explícita do resultado esperado (exemplo refatorado do catálogo, sem usar `assert True`). A asserção usa só a parte fixa da mensagem de `boltons/mathutils.py`, sem o valor de x.

**Métricas do teste refatorado:**

| Métrica                                            | Antes | Depois |
| -------------------------------------------------- | ----- | ------ |
| Complexidade ciclomática da função (cc_por_funcao) | 1     | 2      |
| SLOC do arquivo de teste (raw_por_arquivo_e_total) | 110   | 111    |

#### [R34](https://github.com/harielfr/boltons/commit/f0e33cb) — Unknown Test (tests/test_cacheutils.py::test_lri_cache_eviction)

**Antes:**

```python
    test_cache["key3"] = "value3"
```

**Depois:**

```python
    test_cache["key3"] = "value3"
    assert dict(test_cache) == {"key2": "value2", "key3": "value3"}, \
        "LRI(2) should evict key1 and keep only the two newest keys"
```

**Por que era um test smell:** o teste de regressão só garantia que nenhuma exceção era lançada. Uma política de remoção errada (descartar key2 em vez de key1) passaria.

**Estratégia do CATSPY aplicada:** adicionar a asserção que valida o estado esperado, com mensagem explicativa, como no exemplo do catálogo.

| Métrica                                            | Antes | Depois |
| -------------------------------------------------- | ----- | ------ |
| Complexidade ciclomática da função (cc_por_funcao) | 1     | 2      |
| SLOC do arquivo de teste (raw_por_arquivo_e_total) | 348   | 344    |

#### [R17](https://github.com/harielfr/boltons/commit/3b1d48a) — Verbose Test (tests/test_funcutils_fb.py::test_wraps_injected)

**Antes:**

```python
def test_wraps_injected():
    def inject_string(func):
        @wraps(func, injected="a")
        ...
    assert inject_string(wrappable_func)(2) == (1, 2)
```

**Depois:**

```python
def test_wraps_injected_string_arg():
    assert _inject_string(wrappable_func)(2) == (1, 2)

def test_wraps_injected_nonexistent_arg_raises():
    with pytest.raises(ValueError) as excinfo:
        _inject_nonexistent_arg(wrappable_func)
    assert excinfo.value.arg_name == "X"
```

**Por que era um test smell:** um único teste de cerca de 49 linhas definia 6 decoradores aninhados e verificava 5 cenários diferentes de `wraps(..., injected=...)`, misturando várias responsabilidades.

**Estratégia do CATSPY aplicada:** dividir em métodos menores e focados (exemplo `test_operations` → `test_add_items` e `test_remove_items`). O teste virou 5 (`_string_arg`, `_list_arg`, `_nonexistent_arg_raises`, `_into_varkw`, `_without_varkw_raises`), com os decoradores extraídos para helpers de módulo sem `assert`. Adaptação: a primeira versão deixou dois testes só com `pytest.raises`, que o TEMPY conta como Unknown Test (2 smells novos), e foi rejeitada antes do commit. Na refação, cada exceção ganhou uma asserção real sobre o atributo `arg_name` de `MissingArgument`. O helper morto `inject_missing_argument` (definido e nunca chamado) foi removido.

**Métricas do teste refatorado:**

| Métrica                                            | Antes | Depois                          |
| -------------------------------------------------- | ----- | ------------------------------- |
| Complexidade ciclomática da função (cc_por_funcao) | 4     | 2 em cada um dos 5 testes novos |
| SLOC do arquivo de teste (raw_por_arquivo_e_total) | 211   | 212                             |

#### [R38](https://github.com/harielfr/boltons/commit/c624b10) — Verbose Test (tests/test_dictutils.py::test_one_to_one)

**Antes:** (58 linhas, um único teste)

```python
def test_one_to_one():
    e = OneToOne({1:2})
    def ck(val, inv):
        assert (e, e.inv) == (val, inv)
    ck({1:2}, {2:1})
    e[2] = 3
    ...  # setitem, delitem, copy, pop, setdefault, update, sobreposição, unique
```

**Depois:** (8 testes focados, por exemplo)

```python
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

**Estratégia do CATSPY aplicada:** dividir em métodos menores, um por responsabilidade, cada um com seu próprio estado inicial. Adaptação: a primeira versão usava um helper de asserção, e o TEMPY acusou 7 smells novos (1 Undefined Test e 6 Unknown Test), porque não enxerga asserções dentro de funções auxiliares. A versão final usa asserções explícitas.

| Métrica                                            | Antes        | Depois                    |
| -------------------------------------------------- | ------------ | ------------------------- |
| Complexidade ciclomática da função (cc_por_funcao) | 6 (1 função) | 2 a 7 (8 funções; máx. 7) |
| SLOC do arquivo de teste (raw_por_arquivo_e_total) | 457          | 459                       |

#### [R19](https://github.com/harielfr/boltons/commit/ab72792) — Conditional Test Logic (tests/test_socketutils.py::test_multibyte_delim)

**Antes:**

```python
@pytest.mark.parametrize('with_delim', (True, False))
def test_multibyte_delim(with_delim):
    delim = b'\r\n'
    if with_delim:
        cond_delim = b'\r\n'
    else:
        cond_delim = b''
```

**Depois:**

```python
@pytest.mark.parametrize('with_delim, cond_delim',
                         [(True, b'\r\n'), (False, b'')],
                         ids=['True', 'False'])
def test_multibyte_delim(with_delim, cond_delim):
    delim = b'\r\n'
```

**Por que era um test smell:** o `if/else` no corpo só traduzia um parâmetro em outro valor, colocando lógica condicional dentro do teste. (Antes, o R18 (`9a1f762`) já tinha trocado o laço `for with_delim in (True, False)` por `@pytest.mark.parametrize`.)

**Estratégia do CATSPY aplicada:** cada teste cobre um caso, com os dados definidos fora do corpo, usando `@pytest.mark.parametrize`. Adaptação: foram definidos `ids` explícitos para os nomes dos testes (`test_multibyte_delim[True]` e `[False]`) não mudarem.

**Métricas do teste refatorado:**

| Métrica                                            | Antes | Depois |
| -------------------------------------------------- | ----- | ------ |
| Complexidade ciclomática da função (cc_por_funcao) | 13    | 11     |
| SLOC do arquivo de teste (raw_por_arquivo_e_total) | 313   | 291    |

#### [R39](https://github.com/harielfr/boltons/commit/7ced183) e [R40](https://github.com/harielfr/boltons/commit/175e86f) — Conditional Test Logic (tests/test_cacheutils.py::test_lri)

**Antes:**

```python
    for idx, char in enumerate(string.ascii_letters):
        x = bc[char]
        assert x == char.upper()
        least_recent_insert_index = idx - cache_size
        if least_recent_insert_index >= 0:
            assert len(bc) == cache_size
            for char in string.ascii_letters[least_recent_insert_index+1:idx]:
                assert char in bc
```

**Depois:** (R39 + R40)

```python
@pytest.mark.parametrize("lookup_count", range(1, len(string.ascii_letters) + 1))
def test_lri_on_miss_keeps_most_recent_inserts(lookup_count):
    cache_size = 10
    bc = LRI(cache_size, on_miss=lambda k: k.upper())
    looked_up = string.ascii_letters[:lookup_count]
    assert [bc[char] for char in looked_up] == list(looked_up.upper())
    assert set(bc) == set(looked_up[-cache_size:])
```

**Por que era um test smell:** o `if` deixava os 10 primeiros passos sem verificação de conteúdo, e os laços aninhados escondiam qual passo falhou. A função tinha CC 12.

**Estratégia do CATSPY aplicada:** extrair cada caso para um teste separado. O R39 substituiu o `if` e o `for` interno por uma asserção única, válida em todos os passos; o R40 transformou o `for` externo em `parametrize` (52 casos). O teste ficou mais rigoroso: compara o conjunto exato, então também pega chaves sobrando.

| Métrica                                            | Antes | Depois                                          |
| -------------------------------------------------- | ----- | ----------------------------------------------- |
| Complexidade ciclomática de `test_lri`             | 12    | 9 (R39), depois 7 (R40); teste parametrizado: 3 |
| SLOC do arquivo de teste (raw_por_arquivo_e_total) | 348   | 344                                             |

#### [R20](https://github.com/harielfr/boltons/commit/989f837) — Conditional Test Logic (tests/test_setutils.py::test_iset_index_method)

**Antes:**

```python
    indexed_list = IndexedSet()

    for i in original_list:
        indexed_list.add(i)
```

**Depois:**

```python
    indexed_list = IndexedSet()

    indexed_list.update(original_list)
```

**Por que era um test smell:** o laço só preparava os dados, colocando lógica de controle no corpo do teste sem verificar nada naquele trecho.

**Estratégia do CATSPY aplicada:** tirar a lógica de controle do corpo do teste. O exemplo do catálogo (um teste por ramo) não se aplicava, porque o laço não tinha ramos nem asserções; foi usada uma chamada única a `IndexedSet.update`, que em `boltons/setutils.py` chama `add` item a item, na mesma ordem.

**Métricas do teste refatorado:**

| Métrica                                            | Antes | Depois |
| -------------------------------------------------- | ----- | ------ |
| Complexidade ciclomática da função (cc_por_funcao) | 16    | 15     |
| SLOC do arquivo de teste (raw_por_arquivo_e_total) | 251   | 250    |

## 4. Estado final e comparação

Fonte: `metrics-before-*` e `metrics-after-*`, gerados uma vez na branch `integracao` (as duas branches juntas, antes do rebase do PR).

| Métrica                                                | Antes | Depois | Variação   |
| ------------------------------------------------------ | ----- | ------ | ---------- |
| Total de test smells (TEMPY)                           | 207   | 157    | −50 (−24%) |
| Testes aprovados                                       | 525   | 594    | +69        |
| Testes com falha                                       | 0     | 0      | 0          |
| Cobertura total (%)                                    | 67,37 | 67,38  | +0,01 p.p. |
| SLOC dos testes                                        | 4600  | 4578   | −22        |
| Complexidade ciclomática média das funções de teste    | 3,57  | 3,53   | −0,05      |
| Índice de manutenibilidade médio dos arquivos de teste | 48,20 | 48,06  | −0,14      |

Complementos: 1 teste `xfail` novo (R10) e 12 subtests nas duas medições; 595 testes coletados no final. O código-fonte (`boltons/`) ficou idêntico em todas as métricas (SLOC 8169, MI 41,77, CC média 3,18). Outras colunas do Radon nos testes: linhas físicas 6518 → 6518, linhas lógicas 4287 → 4267, comentários 341 → 339, linhas em branco 1478 → 1501, funções 530 → 540, soma da CC 1894 → 1904.

| Test smell                  | Antes   | Depois  | Variação |
| --------------------------- | ------- | ------- | -------- |
| Conditional Test Logic      | 107     | 100     | −7       |
| Exception Handling          | 29      | 15      | −14      |
| Unknown Test                | 24      | 16      | −8       |
| Redundant Print             | 16      | 9       | −7       |
| Non-Functional Statement    | 14      | 5       | −9       |
| Verbose Test                | 11      | 9       | −2       |
| Programming Paradigms Blend | 3       | 3       | 0        |
| Undefined Test              | 2       | 0       | −2       |
| Sleepy Test                 | 1       | 0       | −1       |
| **Total**                   | **207** | **157** | **−50**  |

## 5. Análise dos resultados

**Quais métricas melhoraram e quais pioraram?** Os smells caíram 24% (207 para 157), o SLOC dos testes caiu 22 linhas e o CC médio das funções de teste caiu de 3,57 para 3,53. Duas métricas pioraram um pouco: o índice de manutenibilidade médio dos arquivos de teste caiu 0,14 ponto (48,20 para 48,06), porque dividir testes e parametrizar aumenta o volume de Halstead dos arquivos (o MI é calculado por arquivo, não por função); e a soma de CC subiu de 1894 para 1904, com 10 funções a mais. A queda maior é de Matheus em `test_lri` (CC 12 para 7) e `test_lru_basic` (26 para 20). Em várias refatorações a CC da função subiu 1 (R22, R34, R35, R37), porque o Radon conta cada `assert` como ponto de decisão; nesses casos o aumento reflete mais verificação, não mais complexidade. Os dois tipos que sumiram por completo são Undefined Test e Sleepy Test.

**Alguma refatoração introduziu novos test smells? Quais?** Nas versões commitadas, uma vez: o R04 (`b923cec`) trocou um `print` por `pass` num `except`, o que criou um Non-Functional Statement; ele foi removido no R21 (`6aa6b6b`), junto com o Exception Handling do mesmo bloco. Duas versões foram descartadas antes do commit: a primeira do R17 de Enzo (2 Unknown Test, por deixar dois testes só com `pytest.raises`) e a primeira do R38 de Matheus (1 Undefined Test e 6 Unknown Test, por usar um helper de asserção que o TEMPY não enxerga). Na comparação final, nenhum tipo de smell aumentou. Fora do que o TEMPY detecta, R22 a R24 comparam strings de `repr`, o que lembra Sensitive Equality, mas ali o `__repr__` é o comportamento testado.

**O resultado dos testes ou a cobertura mudou?** Nenhum teste passou a falhar: 525 passed antes, 594 passed e 1 xfailed depois. O aumento de 69 testes vem de Matheus (+64: 52 casos em `test_lri_on_miss_keeps_most_recent_inserts`, +4 em `test_clear`, +7 da divisão de `test_one_to_one`, +1 do teste extraído de `_multi_sublist_bl`) e de Enzo (+5: `test_wraps_injected` virou 5 testes, +4, e `test_multibyte_delim` virou 2 casos parametrizados, +1). O `xfail` é o R10: ao rodar o `_test_bad_utf8` que nunca era coletado, ele falha de verdade (`to_text()` decodifica `%00` e `%FF`). A cobertura praticamente não mudou (67,37% para 67,38%; 5181 de 7326 linhas nas duas medições), como esperado para mudanças só nos testes. Dois endurecimentos de comportamento foram feitos de propósito, com verificação de que a exceção é determinística: R11 e R12 passaram a _exigir_ um `OSError` que o teste antes só tolerava.

**Quais tipos de smell foram mais fáceis e mais difíceis de refatorar?** Mais fáceis: Redundant Print (remover a linha) e Non-Functional Statement/Exception Handling no padrão `try/except/pass/else: assert False`, com substituição direta por `pytest.raises`. Mais difíceis: Sleepy Test (o atraso era o cenário testado, não uma espera desnecessária), Undefined Test (renomear não bastava: no R10 expôs uma falha real e no R30 transformaria uma fábrica em teste), Verbose Test (as duas primeiras versões criaram smells novos por causa de `pytest.raises` sem asserção e de helpers de asserção) e Conditional Test Logic em `test_lri`, com laços aninhados e estado acumulado. Os Unknown Test pediam julgamento: o teste já usava `pytest.raises`, e foi preciso achar uma asserção real e específica (mensagem ou atributo da exceção).

## 6. Uso do CATSPY

**Como você usou o CATSPY durante a atividade?** Antes de cada refatoração, consultamos a página do smell no catálogo (definição, efeitos negativos, detecção e refatoração indicada) e comparamos o trecho do boltons com o exemplo "com problema". Depois usamos o exemplo refatorado como ponto de partida. A lista geral, com os atributos de qualidade afetados, serviu para priorizar os tipos (Seção 3.1). Também consultamos smells vizinhos para checar se a solução criaria outro problema (Sensitive Equality nos R22 a R24, Return in Test no R30).

**Quais partes do catálogo mais ajudaram (definição, efeitos negativos, exemplo com problema, exemplo refatorado)?** O exemplo refatorado, principalmente em Exception Handling (`assertRaises`, que no pytest é `pytest.raises`), Unknown Test (terminar com uma asserção explícita) e Verbose Test (dividir por responsabilidade). A definição ajudou a decidir se o trecho era mesmo o smell (por exemplo, `pass` em `except`), e os efeitos negativos ajudaram a explicar no diário por que cada trecho era um problema. Os exemplos "em outros contextos" ajudaram pouco num projeto de biblioteca.

**Houve casos em que a estratégia do catálogo não se aplicava diretamente ao projeto? Como você resolveu?** Sim:

- **Sleepy Test (R09):** o catálogo sugere sincronizar por estado ou apagar o `sleep`, mas o silêncio era o cenário testado. Trocamos o `sleep` por uma leitura com timeout verificada em `pytest.raises(Timeout)`.
- **Undefined Test (R10 e R30):** renomear com o prefixo `test` faria o R10 falhar (marcado como `xfail(strict=True)`) e faria o R30 coletar uma fábrica que retorna valor (separamos construção e verificação).
- **Non-Functional Statement (R08 e R27):** no R08 o `except` só tolerava a exceção, então `pytest.raises` deixaria o teste mais rígido (usamos `contextlib.suppress`); no R27 o `pass` era corpo de uma classe de exceção (trocado por uma docstring).
- **Exception Handling (R31 e R33):** os `try` eram de limpeza de recurso, e `assertRaises` não se aplica (usamos `TemporaryDirectory` e `with BufferedWriter`).
- **Unknown Test (R35 e R36):** os testes usavam `pytest.raises`, que o TEMPY não conta como asserção; acrescentamos asserções sobre a mensagem da exceção.
- **Redundant Print (R04):** o print era a única instrução de um `except`, então não dava para só apagá-lo.

**O que faltou ou poderia ser melhorado no catálogo?** Exemplos em estilo pytest (funções com `assert`, `pytest.raises`, `parametrize`): quase todos estão em `unittest`, e a maioria dos testes do boltons é pytest. Também faltam o caso de Exception Handling com `try/finally` de limpeza, um alerta de que renomear helpers com parâmetros ou retorno quebra o pytest (Undefined Test), um nível explícito de severidade para priorizar, uma nota de que o limite de Verbose Test do catálogo (mais de 30 linhas) difere do do TEMPY (45), O exemplo refatorado de Unknown Test tem um erro de indentação no código.

## 7. Divisão do trabalho

| Integrante         | Refatorações realizadas (IDs da Seção 3.2) | Quantidade |
| ------------------ | ------------------------------------------ | ---------- |
| Enzo Hariel        | R01 a R21 (20 da lista, mais a extra R21)  | 21         |
| Matheus dos Santos | R22 a R41                                  | 20         |

A divisão foi por arquivo de teste, para os commits não conflitarem no merge. Enzo também integrou as duas branches (`integracao`), gerou as métricas oficiais e preparou o PR.

## 8. Dificuldades e lições aprendidas

**Quais foram as principais dificuldades (ambiente, dependências, entendimento do código, ferramentas, revisão do PR)?**

- **Ferramentas:** o TEMPY detecta Conditional Test Logic por texto (linhas que começam com `if`, `for` ou `while`), não conta `pytest.raises` como asserção e não enxerga asserções dentro de funções auxiliares. Foi preciso conferir cada arquivo após cada mudança para não criar smells novos (R04, R17, R38). Por isso passamos a conferir, na validação, que nenhum tipo de smell aumenta.
- **Entendimento do código:** o R32 exigiu ler `SpooledBytesIO.write` e o histórico do PR #513 para concluir que o `except BlockingIOError` escondia uma regressão. Os R11 e R12 exigiram ler `recv_until` para provar que a exceção é determinística antes de exigi-la. O R10 revelou que `URL.to_text()` não mantém `%00` e `%FF` codificados.
- **Commits atômicos:** em trechos aninhados ou vizinhos (R32/R33, R39/R40, R18/R19, R05 a R08), foi preciso refatorar um por vez, mantendo uma ocorrência por commit, mesmo quando um `pytest.raises` removia dois smells juntos.
- **Ambiente e fluxo:** no Windows, o arquivo temporário precisa ser fechado antes de apagar o diretório (R31), e o Git avisou sobre conversão LF/CRLF, sem efeito no diff. Um comando de terminal foi colado por engano na linha 1 de `tests/test_socketutils.py` durante uma refatoração; foi revertido com `git checkout -- tests/test_socketutils.py` depois de conferir que o diff era só aquela linha.
- **Upstream:** o `mahmoud/boltons` avançou 28 commits além do commit base e alterou alguns desses arquivos de teste. Antes do PR é preciso fazer rebase sobre o `master` do upstream. A parte de Matheus foi testada nessa base sem conflito (684 aprovados, 1 ignorado, 0 falhas). ⚠ _Atualizar com o resultado do rebase completo depois de feito._

**O que você aprendeu com a atividade?** Que um teste pode passar sem verificar nada (Unknown Test, `except: pass`, prints) e que esses são os smells mais perigosos, porque dão falsa sensação de cobertura. Que um detector é um ponto de partida, não um juiz: várias ocorrências pediam um julgamento diferente da receita padrão, e duas refatorações revelaram comportamento real do código (o `xfail` do R10 e o `except` que escondia o bug do R32). Que parametrizar e dividir testes aumenta o número de testes, mas deixa cada falha muito mais fácil de localizar. E que refatorar teste exige provar que o teste continua falhando quando deveria (teste de mutação no R09, repetição de execuções nos R11 e R12).

## 9. Contribuição (Pull Request)

- **PR:** https://github.com/mahmoud/boltons/pull/710, de `harielfr:pr-upstream` para `mahmoud/boltons:master`.
- **Conteúdo:** os 41 commits de refatoração dos arquivos de teste (R01 a R41), reaplicados (cherry-pick) sobre o `master` atual do upstream, que estava 28 commits à frente do nosso commit base. O PR altera só arquivos em `tests/`; o código de `boltons/`, as pastas `metrics-*`, os scripts de extração e os documentos da atividade **não** fazem parte do PR. Um conflito em `tests/test_namedutils.py` foi resolvido mantendo a nossa asserção (R15) e o teste que o upstream adicionou (`test_non_identifier_alnum_name`).
- **Resultado dos testes na base do upstream:** 690 passed, 1 xfailed, 22 subtests passed, 0 falhas. O xfailed é o R10. O número maior de testes (690, contra 594 na nossa base) vem dos testes que o upstream acrescentou nos 28 commits à frente.
- **Descrição do PR:** segue o template do projeto (`.github/PULL_REQUEST_TEMPLATE.md`), com as seções _The change_, _The backstory_ e _AI assistance_. Esta última declara que parte das refatorações foi assistida por IA (Claude Code) e a outra parte foi feita manualmente.
- **Resposta dos mantenedores:** ⚠ _[nenhuma até o momento / descrever quando houver]_.

## 10. Observações gerais

- **Números oficiais:** as métricas desta versão foram geradas uma vez, na branch `integracao` (as duas branches juntas, sobre o commit base). Os números "depois" do relatório individual de Matheus eram só da branch dele e de conferência local (182 smells, 589 testes, 68% de cobertura com branches); os valores oficiais são os das Seções 2 e 4 (157 smells, 594 testes, 67,38%).
- **Contagem:** são 41 refatorações, das quais 40 são as da lista e 1 é a extra (R21), que corrigiu o Non-Functional Statement criado pelo R04. Os smells vizinhos removidos junto (10) não entram nas 40.
- **Dúvidas para o monitor:** (1) quando um `pytest.raises` remove juntos um Non-Functional Statement e um Exception Handling, conta uma ou duas ocorrências? Aqui contamos uma. (2) O diário deve ser um documento só da dupla ou um por integrante? (3) O PR deve ser um só da dupla ou um por integrante? (4) Entrega depois do prazo (29/10 vs 30/10).
- **Artefatos:** os diários e relatórios individuais, os CSVs e relatórios das métricas e os scripts de extração ficam na branch `integracao` do fork, fora do PR.
- **Sugestão para o TEMPY:** contar `pytest.raises` como asserção (ou tratar a exceção esperada como verificação) e seguir chamadas a helpers, para reduzir falsos positivos de Unknown Test.
- ⚠ _Formulários de triagem e pós-teste: cada integrante preenche o seu._
