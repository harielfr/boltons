# Test Smells em projetos OpenSource

**Disciplina:** Qualidade de Software

**Equipe:** Enzo Hariel e Matheus dos Santos

**Projeto:** boltons — [github.com/mahmoud/boltons](https://github.com/mahmoud/boltons) (commit base `4e5faa3d7e4008d89e0d8bf1ea87b6d9a061a16d`)

**Fork:** [github.com/harielfr/boltons](https://github.com/harielfr/boltons)

## Resumo

Diário de bordo único da dupla, com **41 refatorações** (um commit e uma entrada cada), de **8 tipos** de test smell. Os IDs seguem a ordem das entradas: **R01 a R21 são de Enzo** (R21 é uma refatoração extra, que corrigiu um Non-Functional Statement criado pela R04) e **R22 a R41 são de Matheus** (no diário individual dele eram R01 a R20). Cada entrada traz os quatro campos do modelo, o link do smell no CATSPY e o antes/depois do código. Os links dos commits são `https://github.com/harielfr/boltons/commit/<hash>`.

| Tipo                     | Enzo                        | Matheus     | Total  |
| ------------------------ | --------------------------- | ----------- | ------ |
| Redundant Print          | 4 (R01–R04)                 | 3 (R22–R24) | 7      |
| Non-Functional Statement | 4 (R05–R08) + 1 extra (R21) | 5 (R25–R29) | 10     |
| Sleepy Test              | 1 (R09)                     | 0           | 1      |
| Undefined Test           | 1 (R10)                     | 1 (R30)     | 2      |
| Exception Handling       | 2 (R11–R12)                 | 3 (R31–R33) | 5      |
| Unknown Test             | 4 (R13–R16)                 | 4 (R34–R37) | 8      |
| Verbose Test             | 1 (R17)                     | 1 (R38)     | 2      |
| Conditional Test Logic   | 3 (R18–R20)                 | 3 (R39–R41) | 6      |
| **Total**                | **21**                      | **20**      | **41** |

## R01 | Redundant Print | tests/test_socketutils.py::test_socketutils_netstring | commit [7270697](https://github.com/harielfr/boltons/commit/7270697)

**Autor:** Enzo Hariel · **CATSPY:** [Redundant Print](https://cat-spy.vercel.app/smells/Redundant%20Print)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                     |
| ------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Redundant Print em test_socketutils.py::test_socketutils_netstring (linha 275): um `print("running self tests")` logo após a docstring, que só imprimia uma string fixa e não verificava nada. Nenhum smell vizinho foi removido junto.      |
| Minhas principais dificuldades na remoção do code smell são:                   | Sem dificuldades relevantes: o print exibia só uma string literal e nenhuma asserção dependia dele.                                                                                                                                          |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a definição de Redundant Print (um print em teste é redundante porque a avaliação é feita pelas asserções), o efeito negativo (mais complexidade no código de teste) e a refatoração indicada (remover o print). |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Remoção da instrução print(), sem efeito sobre o comportamento verificado; como nenhuma asserção dependia dela, não havia o que transformar em asserção.                                                                                     |

**Testes:** tests/test_socketutils.py com 11 passed antes e depois (3 execuções, sem instabilidade); TEMPY do arquivo de 47 para 46 smells, sem smell novo; commit 7270697.

Antes:

```python
    """A holistic feature test of BufferedSocket via the NetstringSocket
    wrapper. Runs
    """
    print("running self tests")

    # Set up server
```

Depois:

```python
    """A holistic feature test of BufferedSocket via the NetstringSocket
    wrapper. Runs
    """

    # Set up server
```

## R02 | Redundant Print | tests/test_socketutils.py::test_socketutils_netstring | commit [76ef5c7](https://github.com/harielfr/boltons/commit/76ef5c7)

**Autor:** Enzo Hariel · **CATSPY:** [Redundant Print](https://cat-spy.vercel.app/smells/Redundant%20Print)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                                                    |
| ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Redundant Print em test_socketutils.py::test_socketutils_netstring (linha base 300, linha 299 após o commit da nº 1): um `print("netstring ping-pong latency", dur, "ms")` que só exibia no console a latência medida do ping-pong e não verificava nada. Nenhum smell vizinho foi removido junto.                                                          |
| Minhas principais dificuldades na remoção do code smell são:                   | Sem dificuldades relevantes, porque nenhuma asserção dependia do print. Só foi preciso localizar a linha atual, que mudou 1 linha por causa do commit anterior. O print não virou asserção porque mostrava uma medida de tempo, e uma asserção sobre ela deixaria o teste instável; com isso, a variável `dur` ficou sem uso, e nenhum smell novo apareceu. |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a definição de Redundant Print (um print em teste é redundante porque a avaliação é feita pelas asserções), o efeito negativo (mais complexidade no código de teste) e a refatoração indicada (remover o print).                                                                                                                |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Remoção da instrução print(), sem mudar o comportamento verificado pelo teste.                                                                                                                                                                                                                                                                              |

**Testes:** tests/test_socketutils.py com 11 passed antes, depois e nas 3 execuções de validação, sem instabilidade; TEMPY do arquivo de 46 para 45 smells (Redundant Print de 12 para 11), sem smell novo; commit 76ef5c7; veredito do validador: APROVADO. A suíte completa não foi rodada nesta ocorrência.

Antes:

```python
    for i in range(1000):
        client.write_ns(b'ping')
        assert client.read_ns() == b'pong'
    dur = time.time() - s
    print("netstring ping-pong latency", dur, "ms")
```

Depois:

```python
    for i in range(1000):
        client.write_ns(b'ping')
        assert client.read_ns() == b'pong'
    dur = time.time() - s
```

## R03 | Redundant Print | tests/test_socketutils.py::test_socketutils_netstring | commit [06d64ae](https://github.com/harielfr/boltons/commit/06d64ae)

**Autor:** Enzo Hariel · **CATSPY:** [Redundant Print](https://cat-spy.vercel.app/smells/Redundant%20Print)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                        |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Redundant Print em test_socketutils.py::test_socketutils_netstring (linha base 312, linha 310 após os commits das nº 1 e 2): um `print("netstring pipelined ping-pong latency", dur, "ms")` que só exibia no console a latência do ping-pong em pipeline e não verificava nada. Nenhum smell vizinho foi removido junto.        |
| Minhas principais dificuldades na remoção do code smell são:                   | Sem dificuldades relevantes, porque o print não alimentava nenhuma asserção; bastou confirmar a linha correta, deslocada pelos commits anteriores, e removê-lo. O print não virou asserção porque o valor depende do tempo de execução, e a variável `dur = e - s` ficou sem uso, mantida no lugar. Nenhum smell novo apareceu. |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a definição de Redundant Print (um print em teste é redundante porque o resultado é avaliado pelas asserções), o efeito negativo (aumenta a complexidade do código de teste) e a refatoração indicada (remover o print).                                                                            |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Remoção da instrução print(), mantendo iguais as asserções vizinhas (`all(r == b'pong' ...)` e `getrecvbuffer() == b''`).                                                                                                                                                                                                       |

**Testes:** tests/test_socketutils.py com 11 passed antes, depois e nas 3 execuções de validação, sem instabilidade; TEMPY do arquivo de 45 para 44 smells (Redundant Print de 11 para 10), sem smell novo; commit 06d64ae; veredito do validador: APROVADO. A suíte completa não foi rodada nesta ocorrência.

Antes:

```python
    assert all([r == b'pong' for r in resps])
    assert client.bsock.getrecvbuffer() == b''
    dur = e - s
    print("netstring pipelined ping-pong latency", dur, "ms")
```

Depois:

```python
    assert all([r == b'pong' for r in resps])
    assert client.bsock.getrecvbuffer() == b''
    dur = e - s
```

## R04 | Redundant Print | tests/test_socketutils.py::test_socketutils_netstring | commit [b923cec](https://github.com/harielfr/boltons/commit/b923cec)

**Autor:** Enzo Hariel · **CATSPY:** [Redundant Print](https://cat-spy.vercel.app/smells/Redundant%20Print)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Redundant Print em test_socketutils.py::test_socketutils_netstring (linha base 320, linha 317 após os commits das nº 1 a 3): o `print("raised ConnectionClosed correctly")` era a única instrução de um `except ConnectionClosed:` e só exibia uma mensagem no console. A verificação real é feita pelo próprio try/except, pois se ConnectionClosed não for lançada, `raise Exception('read from closed socket')` faz o teste falhar. |
| Minhas principais dificuldades na remoção do code smell são:                   | O print era a única instrução do bloco except, então não dava para só apagá-lo, e foi preciso colocar um `pass` no lugar. Esse `pass` criou um Non-Functional Statement novo (linha 317), que não foi corrigido porque converter o bloco para pytest.raises corrigiria um Exception Handling que está fora da minha lista. Esse `pass` foi corrigido depois, no R21 (commit `6aa6b6b`).                                                |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a definição de Redundant Print (o resultado do teste vem das asserções, não da saída no console) e a refatoração indicada (remover os prints desnecessários). O exemplo de levar a mensagem para a asserção não se aplicava, porque não há asserção nesse trecho.                                                                                                                                          |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Remoção do print(), com `pass` no lugar para manter o bloco except válido; o try/except foi mantido sem alteração.                                                                                                                                                                                                                                                                                                                     |

**Testes:** tests/test_socketutils.py com 11 passed antes, depois e nas 3 execuções de validação; TEMPY do arquivo com total de 44 para 44 smells (Redundant Print de 10 para 9, Non-Functional Statement de 4 para 5, o novo na linha 317); commit b923cec; veredito do validador: APROVADO.

Antes:

```python
    try:
        client.read_ns()
        raise Exception('read from closed socket')
    except ConnectionClosed:
        print("raised ConnectionClosed correctly")
```

Depois:

```python
    try:
        client.read_ns()
        raise Exception('read from closed socket')
    except ConnectionClosed:
        pass
```

## R05 | Non-Functional Statement | tests/test_socketutils.py::test_short_lines | commit [e880ae3](https://github.com/harielfr/boltons/commit/e880ae3)

**Autor:** Enzo Hariel · **CATSPY:** [Non-Functional Statement](https://cat-spy.vercel.app/smells/Non-Functional%20Statement)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Non-Functional Statement em test_socketutils.py::test_short_lines (linha 33): um `pass` sem efeito dentro de `except ConnectionClosed:`, num try/except/else que só existia para exigir que `bs.recv_size(1)` lançasse ConnectionClosed (o else tinha `assert False, 'expected ConnectionClosed'`). A refatoração removeu junto o Exception Handling vizinho do mesmo método (linha 30), que era o try/except que continha o `pass`. |
| Minhas principais dificuldades na remoção do code smell são:                   | Sem dificuldades relevantes: o `pass` só engolia a exceção esperada e o else já fazia o papel do pytest.raises, então bastou confirmar que o comportamento exigido continua igual. Não surgiu smell novo.                                                                                                                                                                                                                            |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a seção de Non-Functional Statement (definição, efeitos e a nota sobre `pass` dentro de `except`) e a de Exception Handling (definição, refatoração por assertRaises, equivalente a pytest.raises no pytest, e exemplo refatorado).                                                                                                                                                                      |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Substituir o try/except/else por `with pytest.raises(ConnectionClosed):`, sem `match=`, porque o teste original não verificava a mensagem da exceção.                                                                                                                                                                                                                                                                                |

**Testes:** tests/test_socketutils.py com 11 passed antes, depois e nas 3 execuções de validação; TEMPY do arquivo com total de 44 para 42 smells (Non-Functional Statement de 5 para 4, Exception Handling de 16 para 15), sem smell novo; commit e880ae3; veredito do validador: APROVADO.

Antes:

```python
        try:
            bs.recv_size(1)
        except ConnectionClosed:
            pass
        else:
            assert False, 'expected ConnectionClosed'
```

Depois:

```python
        with pytest.raises(ConnectionClosed):
            bs.recv_size(1)
```

## R06 | Non-Functional Statement | tests/test_socketutils.py::test_client_disconnecting | commit [76313f8](https://github.com/harielfr/boltons/commit/76313f8)

**Autor:** Enzo Hariel · **CATSPY:** [Non-Functional Statement](https://cat-spy.vercel.app/smells/Non-Functional%20Statement)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Non-Functional Statement em test_socketutils.py::test_client_disconnecting (linha base 127, 123 após os commits anteriores): um `pass` sem efeito dentro de `except OSError:`, num try/except/else que só existia para exigir que `bx.recv(1)` em socket fechado lançasse OSError (o else tinha `assert False, 'expected socket.error on closed recv'`). A refatoração removeu junto o Exception Handling vizinho do mesmo método (linha 120 em HEAD, cerca de 124 no commit base), que era o try/except que continha o `pass`. |
| Minhas principais dificuldades na remoção do code smell são:                   | Foi preciso confirmar se o except só tolerava a exceção ou se o teste a exigia; o `else: assert False` mostrou que ela era exigida, então pytest.raises preserva o comportamento. O método tem outra ocorrência da lista (nº 7), que não podia ser tocada neste commit. Não surgiu smell novo.                                                                                                                                                                                                                                  |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a seção de Non-Functional Statement (definição e a nota sobre `pass` dentro de `except`) e a de Exception Handling (definição, refatoração com `with pytest.raises(...)` e exemplo refatorado com assertRaises).                                                                                                                                                                                                                                                                                    |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Substituir o try/except/else por `with pytest.raises(OSError):`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                |

**Testes:** tests/test_socketutils.py com 11 passed antes, depois e nas 3 execuções de validação; TEMPY do arquivo com total de 42 para 40 smells (Non-Functional Statement de 4 para 3, Exception Handling de 15 para 14), sem smell novo; commit 76313f8; veredito do validador: APROVADO.

Antes:

```python
    try:
        bx.recv(1)
    except OSError:
        pass
    else:
        assert False, 'expected socket.error on closed recv'
```

Depois:

```python
    with pytest.raises(OSError):
        bx.recv(1)
```

## R07 | Non-Functional Statement | tests/test_socketutils.py::test_client_disconnecting | commit [7b72f5c](https://github.com/harielfr/boltons/commit/7b72f5c)

**Autor:** Enzo Hariel · **CATSPY:** [Non-Functional Statement](https://cat-spy.vercel.app/smells/Non-Functional%20Statement)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                                                                                                       |
| ------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Non-Functional Statement em test_socketutils.py::test_client_disconnecting (linha base 156, linha 148 após os commits anteriores): um `pass` sem efeito dentro de `except OSError:`, num try/except/else que só servia para exigir que `by.send(b'123')` em socket fechado lançasse OSError. A refatoração removeu junto o Exception Handling vizinho (linha 145 em HEAD), o try/except que continha o `pass`. |
| Minhas principais dificuldades na remoção do code smell são:                   | Sem dificuldades relevantes: o `else: assert False` deixava claro que a exceção era obrigatória, então pytest.raises mantém o comportamento, no mesmo padrão da nº 6. Não surgiu smell novo.                                                                                                                                                                                                                   |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a seção de Non-Functional Statement (definição e a nota sobre `pass` dentro de `except`) e a de Exception Handling (definição, refatoração indicada com assertRaises/pytest.raises e exemplo antes/depois).                                                                                                                                                                        |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Substituir o try/except/else por `with pytest.raises(OSError):`, sem `match=` porque o teste original não conferia a mensagem.                                                                                                                                                                                                                                                                                 |

**Testes:** tests/test_socketutils.py com 11 passed antes, depois e nas 3 execuções de validação; TEMPY do arquivo com total de 40 para 38 smells (Non-Functional Statement de 3 para 2, Exception Handling de 14 para 13), sem smell novo; commit 7b72f5c; veredito do validador: APROVADO.

Antes:

```python
    try:
        by.send(b'123')
    except OSError:
        pass
    else:
        assert False, 'expected socket.error on closed send'
```

Depois:

```python
    with pytest.raises(OSError):
        by.send(b'123')
```

## R08 | Non-Functional Statement | tests/test_socketutils.py::test_split_delim | commit [5ae6e58](https://github.com/harielfr/boltons/commit/5ae6e58)

**Autor:** Enzo Hariel · **CATSPY:** [Non-Functional Statement](https://cat-spy.vercel.app/smells/Non-Functional%20Statement)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Non-Functional Statement em test_socketutils.py::test_split_delim (linha base 175, linha 163 após os commits anteriores): um `pass` sem efeito dentro de `except Timeout:`, num try/except que só tolerava o Timeout de `bs.recv_until(delim, timeout=0.0001)`, sem exigir a exceção. A refatoração removeu junto o Exception Handling vizinho (linha 160 em HEAD, cerca de 172 no commit base), o try/except que continha o `pass`. |
| Minhas principais dificuldades na remoção do code smell são:                   | O except tolerava a exceção em vez de exigi-la (não havia `else: assert False`), então o padrão pytest.raises usado nas nº 5 a 7 deixaria o teste mais rígido e mudaria o que ele exige. Foi preciso escolher outra forma, `contextlib.suppress`, que custou uma linha de import a mais. Não surgiu smell novo.                                                                                                                      |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a seção de Non-Functional Statement (definição, as refatorações sugeridas, remover ou adicionar asserção, e a nota sobre `pass` dentro de `except`) e a de Exception Handling (detecção por bloco try/except). O exemplo refatorado com pytest.raises não foi aplicado, por mudar o que o teste exige.                                                                                                   |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Substituir o try/except/pass por `with contextlib.suppress(Timeout):`, acrescentando `import contextlib` no topo do arquivo.                                                                                                                                                                                                                                                                                                         |

**Testes:** tests/test_socketutils.py com 11 passed antes, depois e nas 3 execuções de validação; TEMPY do arquivo com total de 38 para 36 smells (Non-Functional Statement de 2 para 1, Exception Handling de 13 para 12), sem smell novo; commit 5ae6e58; veredito do validador: APROVADO.

Antes:

```python
    y.sendall(first)
    try:
        bs.recv_until(delim, timeout=0.0001)
    except Timeout:
        pass
    y.sendall(second)
```

Depois:

```python
    y.sendall(first)
    with contextlib.suppress(Timeout):
        bs.recv_until(delim, timeout=0.0001)
    y.sendall(second)
```

## R09 | Sleepy Test | tests/test_socketutils.py::test_socketutils_netstring_timeout | commit [d8226a1](https://github.com/harielfr/boltons/commit/d8226a1)

**Autor:** Enzo Hariel · **CATSPY:** [Sleepy Test](https://cat-spy.vercel.app/smells/Sleepy%20Test)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Sleepy Test em test_socketutils.py::test_socketutils_netstring_timeout (linha base 420, linha 404 após os commits anteriores): o teste usava `time.sleep(0.02)` para deixar o cliente calado por mais de 10 ms e provar que o servidor (timeout padrão de 0.01 s, lendo com `read_ns(1)`) respeita o timeout de 1 s da leitura. A espera dependia de um tempo fixo e nada verificava se ela de fato ocorreu.                                                          |
| Minhas principais dificuldades na remoção do code smell são:                   | O sleep não podia ser só apagado, porque o silêncio do cliente é a própria condição exercitada pelo teste, e sincronizar com um evento exigiria mexer no servidor auxiliar, fora da ocorrência. Para confirmar que o teste ainda verifica o timeout, fiz um teste de mutação numa cópia (trocar `read_ns(1)` por `read_ns()` no servidor): o teste refatorado falhou, como o original, e sem a espera passaria. Não surgiu smell novo nem foi removido smell vizinho. |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a seção de Sleepy Test: a definição (pausas explícitas tornam o teste lento e pouco confiável), a detecção (`time.sleep()` dentro do método) e a orientação de refatoração (sincronizar pelo estado ou pela conclusão da tarefa, e não por tempo fixo). Não segui o exemplo que só apaga o sleep, porque aqui o atraso é o cenário testado.                                                                                               |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Substituir o `time.sleep` por uma leitura com timeout verificada, `with pytest.raises(Timeout): client.read_ns(timeout=0.02)`. O silêncio de pelo menos 20 ms passa a vir do próprio mecanismo de timeout, e o `pytest.raises` exige que a leitura expire.                                                                                                                                                                                                            |

**Testes:** tests/test_socketutils.py com 11 passed antes (T1), após a edição (3 execuções) e nas 3 execuções de validação, sem instabilidade; TEMPY do arquivo com total de 36 para 35 smells (Sleepy Test de 1 para 0), sem smell novo; commit d8226a1; veredito do validador: APROVADO.

Antes:

```python
    client = client_connect()
    time.sleep(0.02)
    client.write_ns(b'ping')
    assert client.read_ns() == b'pong'
```

Depois:

```python
    client = client_connect()
    with pytest.raises(Timeout):
        client.read_ns(timeout=0.02)
    client.write_ns(b'ping')
    assert client.read_ns() == b'pong'
```

## R10 | Undefined Test | tests/test_urlutils.py::\_test_bad_utf8 | commit [3ca263c](https://github.com/harielfr/boltons/commit/3ca263c)

**Autor:** Enzo Hariel · **CATSPY:** [Undefined Test](https://cat-spy.vercel.app/smells/Undefined%20Test)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Undefined Test em test_urlutils.py::\_test_bad_utf8 (linha 182): a função tinha asserção, mas o nome começava com `_test`, então o pytest não a coletava. Ela não era chamada em lugar nenhum e nunca rodava, dando a falsa impressão de que o comportamento estava coberto.                                                                                                                                                                             |
| Minhas principais dificuldades na remoção do code smell são:                   | Ao ser executada, a função falha: `URL('http://xn--9ca.com/%00%FF/%C3%A9').to_text()` devolve `'http://é.com/\x00�/é'`, e o teste espera `'http://é.com/%00%FF/é'`. O renomeio simples colocaria uma falha nova na suíte, e não era permitido alterar `boltons/` nem o valor esperado; por isso marquei o teste com `xfail(strict=True)`, que documenta a falha e acusa se o código for corrigido. Não surgiu smell novo nem foi removido smell vizinho. |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a seção de Undefined Test: a definição (método não reconhecido pelo framework, com falsa impressão de correção), o efeito (falso positivo) e a refatoração indicada (renomear com o prefixo test), além da nota de conferir a assinatura.                                                                                                                                                                                    |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Renomear a função para `test_bad_utf8` (prefixo test), possível porque ela não recebe parâmetros e o pytest não procura fixtures, e marcá-la com `@pytest.mark.xfail(strict=True, reason=...)`. Corpo, valor esperado e asserção ficaram intactos.                                                                                                                                                                                                       |

**Testes:** tests/test_urlutils.py com 127 passed antes (T1) e 127 passed, 1 xfailed após a edição (T2) e na validação (T3); com `--runxfail` o erro real é o AssertionError descrito acima; TEMPY do arquivo com total de 10 para 9 smells (Undefined Test de 1 para 0), sem smell novo; commit 3ca263c; veredito do validador: APROVADO.

Antes:

```python
def _test_bad_utf8():  # not part of the API
    bad_bin_url = 'http://xn--9ca.com/%00%FF/%C3%A9'
    url = URL(bad_bin_url)
```

Depois:

```python
@pytest.mark.xfail(strict=True, reason="to_text() decodifica %00/%FF em vez de mantê-los codificados")
def test_bad_utf8():  # not part of the API
    bad_bin_url = 'http://xn--9ca.com/%00%FF/%C3%A9'
    url = URL(bad_bin_url)
```

## R11 | Exception Handling | tests/test_socketutils.py::test_basic_nonblocking | commit [7c7fbc8](https://github.com/harielfr/boltons/commit/7c7fbc8)

**Autor:** Enzo Hariel · **CATSPY:** [Exception Handling](https://cat-spy.vercel.app/smells/Exception%20Handling)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                                                                                                        |
| ------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Exception Handling em test_socketutils.py::test_basic_nonblocking (linha base 190, linha 177 após os commits anteriores), no 1º `try` do método (bloco "per-call timeout"): o teste usava `try/except OSError` para capturar o erro de `bs.recv_until(delim, timeout=0)` e checar o errno dentro do `except`. Sem um `else: fail`, o teste passaria mesmo que nenhuma exceção ocorresse.                        |
| Minhas principais dificuldades na remoção do code smell são:                   | O `try` original só tolerava a exceção, então o `pytest.raises` deixa o teste mais rígido, pois passa a exigi-la; confirmei que ela é determinística (socketpair vazio e `timeout=0`, com o teste isolado rodando 30 vezes sem falha) e aprovei esse endurecimento. `contextlib.suppress` não serviria, porque não permitiria manter a asserção do errno. Não surgiu smell novo nem foi removido smell vizinho. |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a seção de Exception Handling: a definição (resultado que depende de exceção capturada com try/except), a detecção (try/except dentro do teste) e o exemplo refatorado (assertRaises, que no pytest corresponde a `pytest.raises`).                                                                                                                                                 |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Substituir o `try/except` por `with pytest.raises(OSError) as excinfo:` e manter a checagem do erro como `assert excinfo.value.errno == errno.EWOULDBLOCK`.                                                                                                                                                                                                                                                     |

**Testes:** tests/test_socketutils.py com 11 passed antes (T1), nas 3 execuções após a edição (T2) e nas 3 execuções de validação (T3), mais 30/30 do teste isolado; TEMPY do arquivo com total de 35 para 34 smells (Exception Handling de 12 para 11), sem smell novo; commit 7c7fbc8; veredito do validador: APROVADO.

Antes:

```python
    try:
        bs.recv_until(delim, timeout=0)
    except OSError as se:
        assert se.errno == errno.EWOULDBLOCK
    y.sendall(delim)  # sending an empty message, effectively
```

Depois:

```python
    with pytest.raises(OSError) as excinfo:
        bs.recv_until(delim, timeout=0)
    assert excinfo.value.errno == errno.EWOULDBLOCK
    y.sendall(delim)  # sending an empty message, effectively
```

## R12 | Exception Handling | tests/test_socketutils.py::test_basic_nonblocking | commit [56c55d6](https://github.com/harielfr/boltons/commit/56c55d6)

**Autor:** Enzo Hariel · **CATSPY:** [Exception Handling](https://cat-spy.vercel.app/smells/Exception%20Handling)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Exception Handling em test_socketutils.py::test_basic_nonblocking (linha base 201, linha 187 após os commits anteriores), no segundo `try` do método (bloco "instance-level default timeout"). O teste usava `try/except OSError` para capturar o erro de `bs.recv_until(delim)` com `timeout=0` na instância e checava o errno dentro do `except`; sem `else: fail`, ele passaria mesmo se nenhuma exceção ocorresse.                                                 |
| Minhas principais dificuldades na remoção do code smell são:                   | Como a exceção antes era só tolerada, precisei confirmar que ela é determinística antes de exigi-la: li `recv_until` no código-fonte (`timeout=0` deixa o socket não bloqueante e o buffer está vazio) e rodei o cenário isolado 200 vezes, com a validação repetindo 1000 vezes mais 50 execuções do teste, sempre com BlockingIOError/EWOULDBLOCK. Esse endurecimento já tinha sido aprovado por mim na nº 11. Não surgiu smell novo nem foi removido smell vizinho. |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a seção de Exception Handling: a definição, a detecção (try/except dentro do teste) e a refatoração (assertRaises, que no pytest corresponde a `pytest.raises`).                                                                                                                                                                                                                                                                           |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Substituir o `try/except` por `with pytest.raises(OSError) as excinfo:` e manter a checagem do erro como `assert excinfo.value.errno == errno.EWOULDBLOCK`, no mesmo padrão da nº 11.                                                                                                                                                                                                                                                                                  |

**Testes:** tests/test_socketutils.py com 11 passed antes (T1), nas 3 execuções após a edição (T2) e nas 3 execuções de validação (T3); TEMPY do arquivo com total de 34 para 33 smells (Exception Handling de 11 para 10), sem smell novo; commit 56c55d6; veredito do validador: APROVADO.

Antes:

```python
    try:
        bs.recv_until(delim)
    except OSError as se:
        assert se.errno == errno.EWOULDBLOCK
    y.sendall(delim)
```

Depois:

```python
    with pytest.raises(OSError) as excinfo:
        bs.recv_until(delim)
    assert excinfo.value.errno == errno.EWOULDBLOCK
    y.sendall(delim)
```

## R13 | Unknown Test | tests/test_namedutils.py::test_empty_field_name_raises_value_error | commit [341557d](https://github.com/harielfr/boltons/commit/341557d)

**Autor:** Enzo Hariel · **CATSPY:** [Unknown Test](https://cat-spy.vercel.app/smells/Unknown%20Test)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Unknown Test em test_namedutils.py::test_empty_field_name_raises_value_error (linha 32). O teste só envolvia `factory('Point', ['x', ''])` em `with pytest.raises(ValueError):`, sem nenhuma asserção explícita, e não verificava se o ValueError vinha da validação de nome vazio.                                                                                                                                                                    |
| Minhas principais dificuldades na remoção do code smell são:                   | Como o teste já usava `pytest.raises`, à primeira vista não faltava verificação; foi preciso ler o detector do TEMPY (`how_many_assertions`) para entender que ele só conta linhas com `assert`. Também foi preciso escolher uma asserção real e estável sobre a mensagem, em vez de uma trivial. A validação observou que a mesma mensagem também aparece para caracteres inválidos, então a asserção sozinha não prova que a causa foi o nome vazio. |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a seção de Unknown Test: a definição (teste sem asserções claras), a detecção (método sem asserções explícitas), a refatoração com o exemplo refatorado (terminar com asserção explícita do resultado esperado) e a nota de não usar `assert True`.                                                                                                                                                                        |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Adicionar asserção: capturar a exceção com `as excinfo` e verificar `'must be non-empty' in str(excinfo.value)`, a mensagem fixa lançada por boltons/namedutils.py tanto para `namedtuple` quanto para `namedlist`.                                                                                                                                                                                                                                    |

**Testes:** tests/test_namedutils.py com 9 passed antes (T1), após a edição (T2) e na validação (T3); TEMPY do arquivo com Unknown Test de 3 para 2, sem smell novo e sem smell vizinho removido; commit 341557d; veredito do validador: APROVADO.

Antes:

```python
def test_empty_field_name_raises_value_error(factory):
    ...
    with pytest.raises(ValueError):
        factory('Point', ['x', ''])
```

Depois:

```python
def test_empty_field_name_raises_value_error(factory):
    ...
    with pytest.raises(ValueError) as excinfo:
        factory('Point', ['x', ''])
    assert 'must be non-empty' in str(excinfo.value)
```

## R14 | Unknown Test | tests/test_namedutils.py::test_empty_type_name_raises_value_error | commit [0107779](https://github.com/harielfr/boltons/commit/0107779)

**Autor:** Enzo Hariel · **CATSPY:** [Unknown Test](https://cat-spy.vercel.app/smells/Unknown%20Test)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                   |
| ------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Unknown Test em test_namedutils.py::test_empty_type_name_raises_value_error (linha 40 no commit base, 41 após a nº 13). O teste só envolvia `factory('', ['x', 'y'])` em `with pytest.raises(ValueError):`, sem nenhuma asserção explícita, e não verificava se o erro vinha do nome de tipo vazio.                        |
| Minhas principais dificuldades na remoção do code smell são:                   | O texto fixo da mensagem de erro é o mesmo para nome vazio e para caractere inválido, então foi preciso achar uma asserção que provasse a causa. Resolvi conferindo em boltons/namedutils.py que o valor rejeitado entra no fim da mensagem via `%r` e que nenhum ValueError anterior pode ser lançado para typename `''`. |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a seção de Unknown Test: a definição (teste sem asserção clara), a refatoração com o exemplo refatorado (asserção explícita sobre o resultado esperado) e a nota de não usar `assert True`.                                                                                                    |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Adicionar asserção: capturar a exceção com `as excinfo` e verificar `str(excinfo.value).endswith(': ' + repr(''))`, confirmando que o nome rejeitado na mensagem (igual em `namedtuple` e `namedlist`) é o vazio.                                                                                                          |

**Testes:** tests/test_namedutils.py com 9 passed antes (T1), após a edição (T2) e na validação (T3); TEMPY do arquivo com Unknown Test de 2 para 1, sem smell novo e sem smell vizinho removido; commit 0107779; veredito do validador: APROVADO.

Antes:

```python
def test_empty_type_name_raises_value_error(factory):
    with pytest.raises(ValueError):
        factory('', ['x', 'y'])
```

Depois:

```python
def test_empty_type_name_raises_value_error(factory):
    with pytest.raises(ValueError) as excinfo:
        factory('', ['x', 'y'])
    assert str(excinfo.value).endswith(': ' + repr(''))
```

## R15 | Unknown Test | tests/test_namedutils.py::test_invalid_character_still_raises_value_error | commit [e18cb8d](https://github.com/harielfr/boltons/commit/e18cb8d)

**Autor:** Enzo Hariel · **CATSPY:** [Unknown Test](https://cat-spy.vercel.app/smells/Unknown%20Test)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                                                                          |
| ------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Unknown Test em test_namedutils.py::test_invalid_character_still_raises_value_error (linha 46 no commit base, 48 após as nº 13 e 14). O teste de regressão só envolvia `factory('Point', ['x', 'y-z'])` em `with pytest.raises(ValueError):`, sem nenhuma asserção explícita, e não verificava qual nome era rejeitado.                                                           |
| Minhas principais dificuldades na remoção do code smell são:                   | Foi preciso confirmar em boltons/namedutils.py qual ValueError é lançado para essa entrada. Com `rename=False`, `'Point'` e `'x'` passam em todas as checagens e `'y-z'` falha logo na primeira (mensagem "must be non-empty and can only contain alphanumeric characters and underscores: %r"), igual em `namedtuple` e `namedlist`, então nenhum outro ValueError ocorre antes. |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a seção de Unknown Test: a definição (teste sem asserções claras), a refatoração indicada com o exemplo refatorado (introduzir asserções que validem o resultado esperado) e a nota de não usar `assert True`.                                                                                                                                        |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Adicionar asserção: capturar a exceção com `as excinfo` e verificar `str(excinfo.value).endswith(': ' + repr('y-z'))`, mesmo padrão da nº 14, confirmando que o nome rejeitado é `'y-z'`.                                                                                                                                                                                         |

**Testes:** tests/test_namedutils.py com 9 passed antes (T1), após a edição (T2) e na validação (T3); TEMPY do arquivo com Unknown Test de 1 para 0 (test_namedutils.py ficou sem smells), sem smell novo e sem smell vizinho removido; commit e18cb8d; veredito do validador: APROVADO.

Antes:

```python
def test_invalid_character_still_raises_value_error(factory):
    with pytest.raises(ValueError):
        factory('Point', ['x', 'y-z'])
```

Depois:

```python
def test_invalid_character_still_raises_value_error(factory):
    with pytest.raises(ValueError) as excinfo:
        factory('Point', ['x', 'y-z'])
    assert str(excinfo.value).endswith(': ' + repr('y-z'))
```

## R16 | Unknown Test | tests/test_mathutils.py::test_ceil_oor_upper | commit [a98b4ab](https://github.com/harielfr/boltons/commit/a98b4ab)

**Autor:** Enzo Hariel · **CATSPY:** [Unknown Test](https://cat-spy.vercel.app/smells/Unknown%20Test)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                              |
| ------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Unknown Test em test_mathutils.py::test_ceil_oor_upper (linha 52). O teste só envolvia `ceil(OUT_OF_RANGE_UPPER, OPTIONS)` em `with raises(ValueError):`, sem nenhuma asserção explícita, e não verificava se o erro era o de valor acima de todas as opções de teto. |
| Minhas principais dificuldades na remoção do code smell são:                   | O teste já verificava a exceção com `raises`, mas o TEMPY só conta linhas com `assert`, então foi preciso escolher uma asserção real e específica. Conferi a mensagem lançada em boltons/mathutils.py, sem alterá-lo, e usei só a parte fixa dela (sem o valor de x). |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a seção de Unknown Test: a definição (teste sem asserção clara), a regra de refatoração (introduzir asserção que valide explicitamente o resultado esperado) e o exemplo refatorado.                                                      |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Adicionar asserção: capturar a exceção com `as exc_info` e verificar que a mensagem contém `'no ceil options greater than or equal to'`, único ValueError possível em `ceil` para essa entrada (a mensagem de `floor` é diferente).                                   |

**Testes:** tests/test_mathutils.py com 14 passed antes (T1), após a edição (T2) e na validação (T3); TEMPY do arquivo de 5 para 4 smells (Unknown Test de 3 para 2), sem smell novo e sem smell vizinho removido; commit a98b4ab; veredito do validador: APROVADO.

Antes:

```python
def test_ceil_oor_upper():
    with raises(ValueError):
        ceil(OUT_OF_RANGE_UPPER, OPTIONS)
```

Depois:

```python
def test_ceil_oor_upper():
    with raises(ValueError) as exc_info:
        ceil(OUT_OF_RANGE_UPPER, OPTIONS)
    assert 'no ceil options greater than or equal to' in str(exc_info.value)
```

## R17 | Verbose Test | tests/test_funcutils_fb.py::test_wraps_injected | commit [3b1d48a](https://github.com/harielfr/boltons/commit/3b1d48a)

**Autor:** Enzo Hariel · **CATSPY:** [Verbose Test](https://cat-spy.vercel.app/smells/Verbose%20Test)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Verbose Test em test_funcutils_fb.py::test_wraps_injected (linhas 56 a 104). Um único teste de cerca de 49 linhas definia 6 decoradores aninhados e verificava 5 cenários diferentes de `wraps(..., injected=...)` (string, lista, argumento inexistente, injeção em `**kw` e sem varkw), misturando várias responsabilidades. O helper interno `inject_missing_argument`, definido e nunca chamado (código morto, sem asserção), foi removido sem perda de verificação.                                                                                                                                   |
| Minhas principais dificuldades na remoção do code smell são:                   | A primeira versão da divisão deixou dois testes só com `pytest.raises`, que o TEMPY conta como Unknown Test (2 smells novos), e foi rejeitada; na refação foi preciso achar no código-fonte uma asserção real e específica para cada exceção (o atributo `arg_name` de `MissingArgument`, subclasse de ValueError lançada por `FunctionBuilder.remove_arg` em boltons/funcutils.py). Também foi preciso nomear os helpers sem o prefixo `test` para o pytest não coletá-los.                                                                                                                               |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a seção de Verbose Test (definição: teste longo com várias responsabilidades; refatoração: separar em métodos menores e focados, com dados compartilhados em fixture/setup; exemplo `test_operations` dividido em `test_add_items` e `test_remove_items`) e a de Unknown Test (definição e refatoração: introduzir asserção que valide o resultado, nunca `assert True`).                                                                                                                                                                                                      |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Dividir o teste em 5 testes com uma responsabilidade cada (`test_wraps_injected_string_arg`, `_list_arg`, `_nonexistent_arg_raises`, `_into_varkw`, `_without_varkw_raises`) e extrair os decoradores para helpers de módulo sem assert (`_inject_string`, `_inject_list`, `_inject_nonexistent_arg`, `_inject_misc_argument`, `_inject_misc_argument_no_varkw`), mantendo as 5 asserções originais com as mesmas entradas e valores esperados. Nos testes de exceção, usar `pytest.raises(ValueError) as excinfo` e verificar `excinfo.value.arg_name`; remover o helper morto `inject_missing_argument`. |

**Testes:** tests/test_funcutils_fb.py com 15 passed antes (T1) e 19 passed após a edição (T2) e na validação (T3), já que 1 teste virou 5, cada um passando isolado; TEMPY do arquivo de 5 para 4 smells (Verbose Test de 1 para 0; Unknown Test 4 → 4, os mesmos métodos de antes), sem smell novo; suíte completa (T7) com 529 passed, 1 xfailed, 12 subtests passed; commit 3b1d48a; veredito do validador: APROVADO (após uma refação).

Antes:

```python
def test_wraps_injected():
    def inject_string(func):
        @wraps(func, injected="a")
        ...
    assert inject_string(wrappable_func)(2) == (1, 2)
```

Depois:

```python
def test_wraps_injected_string_arg():
    assert _inject_string(wrappable_func)(2) == (1, 2)

def test_wraps_injected_nonexistent_arg_raises():
    with pytest.raises(ValueError) as excinfo:
        _inject_nonexistent_arg(wrappable_func)
    assert excinfo.value.arg_name == "X"
```

## R18 | Conditional Test Logic | tests/test_socketutils.py::test_multibyte_delim | commit [9a1f762](https://github.com/harielfr/boltons/commit/9a1f762)

**Autor:** Enzo Hariel · **CATSPY:** [Conditional Test Logic](https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                                                   |
| ------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Conditional Test Logic em test_socketutils.py::test_multibyte_delim (linha base 47, linha 44 após os commits anteriores): o laço externo `for with_delim in (True, False):` repetia todo o corpo do teste, misturando dois casos num único teste e escondendo qual deles falhou.                                                                           |
| Minhas principais dificuldades na remoção do code smell são:                   | A nº 19 (o `if with_delim` logo abaixo) está no mesmo método, então foi preciso confirmar que dava para remover só o laço sem tocar no `if`, mantendo um commit por ocorrência. Remover o laço exigiu reindentar o corpo inteiro, o que deixa o diff grande (28+/28-) embora a lógica não mude. Não surgiu smell novo e nenhum smell vizinho foi removido. |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a seção de Conditional Test Logic: a definição (laços e condicionais dentro do teste), a refatoração indicada (cada teste cobre um caso) e a nota de que um laço que repete as mesmas asserções pode virar `@pytest.mark.parametrize`.                                                                                         |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Parametrizar com `@pytest.mark.parametrize('with_delim', (True, False))`, recebendo `with_delim` como parâmetro do teste no lugar do laço; o corpo só foi reindentado (`git diff -w` mostra que a única mudança de lógica é o for → parametrize), e os mesmos casos (2 valores × 4 maxsizes) e asserções continuam cobertos.                               |

**Testes:** tests/test_socketutils.py com 11 passed antes (T1, 3x) e 12 passed após a edição (T2, 3x) e na validação (T3, 3 execuções), já que o teste virou 2 casos parametrizados; TEMPY do arquivo de 33 para 32 smells (Conditional Test Logic de 11 para 10), sem smell novo e sem vizinho removido; suíte completa (T7) com 530 passed, 1 xfailed, 12 subtests passed (+1 caso parametrizado); commit 9a1f762; veredito do validador: APROVADO.

Antes:

```python
def test_multibyte_delim():
    ...
    delim = b'\r\n'
    for with_delim in (True, False):
        if with_delim:
            cond_delim = b'\r\n'
```

Depois:

```python
@pytest.mark.parametrize('with_delim', (True, False))
def test_multibyte_delim(with_delim):
    ...
    delim = b'\r\n'
    if with_delim:
        cond_delim = b'\r\n'
```

## R19 | Conditional Test Logic | tests/test_socketutils.py::test_multibyte_delim | commit [ab72792](https://github.com/harielfr/boltons/commit/ab72792)

**Autor:** Enzo Hariel · **CATSPY:** [Conditional Test Logic](https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                                                     |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Conditional Test Logic em test_socketutils.py::test_multibyte_delim (linha base 48, linha 45 após o commit da nº 18): o teste usava um `if with_delim: ... else: ...` no corpo só para traduzir o parâmetro `with_delim` no delimitador esperado (`cond_delim`), colocando lógica condicional dentro do teste.                                               |
| Minhas principais dificuldades na remoção do code smell são:                   | Sem dificuldades relevantes, porque o `if` só convertia um parâmetro em outro valor. O único cuidado foi definir `ids` explícitos para que os nomes dos testes (`test_multibyte_delim[True]`/`[False]`) não mudassem, já que sem eles o pytest geraria ids a partir dos bytes. Não surgiu smell novo e nenhum smell vizinho foi removido.                    |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a seção de Conditional Test Logic: a definição (condicional dentro do teste), a refatoração indicada (cada teste cobre um caso, com os dados definidos fora do corpo) e a nota de usar `@pytest.mark.parametrize`.                                                                                                               |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Remover o if/else e passar o valor já resolvido pela parametrização: `@pytest.mark.parametrize('with_delim, cond_delim', [(True, b'\r\n'), (False, b'')], ids=['True', 'False'])`. `with_delim` continua como parâmetro porque é usado no corpo (`'with_delimiter': with_delim`), e os mesmos casos (2 valores × 4 maxsizes) e asserções continuam cobertos. |

**Testes:** tests/test_socketutils.py com 12 passed antes (T1) e após a edição (T2, 3x) e na validação (T3, 3 execuções, mesmos nomes coletados); TEMPY do arquivo de 32 para 31 smells (Conditional Test Logic de 10 para 9), sem smell novo e sem vizinho removido; suíte completa (T7) com 530 passed, 1 xfailed, 12 subtests passed; commit ab72792; veredito do validador: APROVADO.

Antes:

```python
@pytest.mark.parametrize('with_delim', (True, False))
def test_multibyte_delim(with_delim):
    delim = b'\r\n'
    if with_delim:
        cond_delim = b'\r\n'
    else:
        cond_delim = b''
```

Depois:

```python
@pytest.mark.parametrize('with_delim, cond_delim',
                         [(True, b'\r\n'), (False, b'')],
                         ids=['True', 'False'])
def test_multibyte_delim(with_delim, cond_delim):
    delim = b'\r\n'
```

## R20 | Conditional Test Logic | tests/test_setutils.py::test_iset_index_method | commit [989f837](https://github.com/harielfr/boltons/commit/989f837)

**Autor:** Enzo Hariel · **CATSPY:** [Conditional Test Logic](https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Conditional Test Logic em test_setutils.py::test_iset_index_method (linha 202): o teste usava um laço `for i in original_list: indexed_list.add(i)` só para preparar os dados, colocando lógica de controle no corpo do teste sem verificar nada nesse trecho.                                                                                                                                                                                                                                                 |
| Minhas principais dificuldades na remoção do code smell são:                   | O método tem complexidade alta (10 ocorrências de Conditional Test Logic), então foi preciso confirmar que o laço da linha 202 era independente (sem `if` aninhado) e podia sair sem mexer nos outros 9. Também verifiquei em boltons/setutils.py que `IndexedSet.update` com um iterável chama `add` item a item, na mesma ordem, deixando o mesmo estado interno; o validador confirmou que `add` continua coberto (via `update` e o construtor). Não surgiu smell novo e nenhum smell vizinho foi removido. |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a seção de Conditional Test Logic: a definição (laços e `if` dentro de testes), a detecção (teste com instruções condicionais) e a ideia de tirar a lógica de controle do corpo do teste. O exemplo refatorado (um teste por ramo) não se aplicava, porque o laço não tinha ramos nem asserções.                                                                                                                                                                                   |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Substituir o laço de preparação por uma única chamada `indexed_list.update(original_list)`, mantendo intactas as asserções seguintes (`index`/`pop`). Não houve parametrização nem extração de helper.                                                                                                                                                                                                                                                                                                         |

**Testes:** tests/test_setutils.py com 16 passed antes (T1), após a edição (T2) e na validação (T3); TEMPY do arquivo de 21 para 20 smells (Conditional Test Logic de 20 para 19; em test_iset_index_method de 10 para 9), sem smell novo e sem vizinho removido; suíte completa (T7) com 530 passed, 1 xfailed, 12 subtests passed; commit 989f837; veredito do validador: APROVADO.

Antes:

```python
    indexed_list = IndexedSet()

    for i in original_list:
        indexed_list.add(i)
```

Depois:

```python
    indexed_list = IndexedSet()

    indexed_list.update(original_list)
```

## R21 | Non-Functional Statement | tests/test_socketutils.py::test_socketutils_netstring | commit [6aa6b6b](https://github.com/harielfr/boltons/commit/6aa6b6b)

**Autor:** Enzo Hariel · **CATSPY:** [Non-Functional Statement](https://cat-spy.vercel.app/smells/Non-Functional%20Statement) · **Refatoração extra (fora das 20 da lista)**

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Non-Functional Statement em test_socketutils.py::test_socketutils_netstring: um `pass` sem efeito dentro de `except ConnectionClosed:`, criado pela refatoração nº 4 (commit b923cec), que trocou um print por pass. O try/except só servia para exigir que `client.read_ns()` lançasse ConnectionClosed depois de `write_ns(b'close')`, com um `raise Exception('read from closed socket')` como sentinela de falha. A refatoração removeu junto o Exception Handling desse try (linha 296 em HEAD), que não fazia parte da lista das 20. |
| Minhas principais dificuldades na remoção do code smell são:                   | Foi preciso confirmar que o teste exigia a exceção, e não apenas a tolerava: como Exception não é subclasse de ConnectionClosed (MRO ConnectionClosed -> Error -> OSError -> Exception), a sentinela fazia o teste falhar sem ConnectionClosed, por isso usei pytest.raises e não contextlib.suppress. O validador comparou as duas versões em 5 cenários (sem exceção, ConnectionClosed, Timeout, NetstringProtocolError, ValueError) e o resultado foi igual em todos. Não surgiu smell novo.                                            |
| Usei o CATSPY para:                                                            | ler, no catálogo do CATSPY, a seção de Non-Functional Statement (definição, a refatoração "adicionar a asserção que o método deveria verificar" e a nota sobre `pass` dentro de `except`) e a seção de Exception Handling (refatoração com pytest.raises e o exemplo refatorado try/self.fail/except -> assertRaises).                                                                                                                                                                                                                     |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Substituir o try/except/pass e a sentinela `raise Exception(...)` por `with pytest.raises(ConnectionClosed): client.read_ns()`.                                                                                                                                                                                                                                                                                                                                                                                                            |

**Testes:** tests/test_socketutils.py com 12 passed na linha de base (T1), após a edição (T2, 3x) e nas 3 execuções da validação (T3); TEMPY do arquivo de 31 para 29 smells (Non-Functional Statement 1 -> 0, Exception Handling 10 -> 9), sem smell novo; suíte completa (T7) com 530 passed, 1 xfailed, 12 subtests passed; commit 6aa6b6b; veredito do validador: APROVADO.

Antes:

```python
    client.write_ns(b'close')
    try:
        client.read_ns()
        raise Exception('read from closed socket')
    except ConnectionClosed:
        pass
```

Depois:

```python
    client.write_ns(b'close')
    with pytest.raises(ConnectionClosed):
        client.read_ns()
```

## R22 | Redundant Print | tests/test_cacheutils.py::test_callable_cached_dec (linha 297) | commit [dac7ea0](https://github.com/harielfr/boltons/commit/dac7ea0)

**Autor:** Matheus dos Santos · **CATSPY:** [Redundant Print](https://cat-spy.vercel.app/smells/Redundant%20Print)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                         |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Redundant Print: `print(repr(func))` no fim do teste, sem nenhuma verificação sobre o valor impresso.                                                                                                                                            |
| Minhas principais dificuldades na remoção do code smell são:                   | Decidir entre apagar o print ou transformá-lo em asserção. O print existia para exercitar o `__repr__` de CachedFunction, então apagar perderia essa intenção. Li `CachedFunction.__repr__` em boltons/cacheutils.py para saber o formato exato. |
| Usei o CATSPY para:                                                            | Definição e exemplo refatorado (substituir o print por uma asserção com a informação relevante).                                                                                                                                                 |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Substituição do print por asserção: `assert repr(func) == f'CachedFunction(func={inner_func!r})'`. Testes do arquivo: 23 aprovados.                                                                                                              |

## R23 | Redundant Print | tests/test_cacheutils.py::test_cachedmethod (linha 356) | commit [19fbfde](https://github.com/harielfr/boltons/commit/19fbfde)

**Autor:** Matheus dos Santos · **CATSPY:** [Redundant Print](https://cat-spy.vercel.app/smells/Redundant%20Print)

| Campo                                                                          | Resposta                                                                                                                                                                     |
| ------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Redundant Print: `print(repr(car_two.door))`, logo abaixo do comentário "always check the repr".                                                                             |
| Minhas principais dificuldades na remoção do code smell são:                   | O método está vinculado a uma instância, e o repr muda nesse caso (inclui `bound_to`). Foi preciso obter a função original por `Car.door.func` para montar o valor esperado. |
| Usei o CATSPY para:                                                            | Definição e exemplo refatorado.                                                                                                                                              |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Substituição do print por asserção sobre o repr do CachedMethod vinculado (`<CachedMethod func=... scoped=False typed=False bound_to=...>`). Um commit só para este print.   |

## R24 | Redundant Print | tests/test_cacheutils.py::test_cachedmethod (linha 357) | commit [b9c420e](https://github.com/harielfr/boltons/commit/b9c420e)

**Autor:** Matheus dos Santos · **CATSPY:** [Redundant Print](https://cat-spy.vercel.app/smells/Redundant%20Print)

| Campo                                                                          | Resposta                                                                                                                                                                                                     |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Redundant Print: `print(repr(Car.door))`, o método acessado pela classe (não vinculado).                                                                                                                     |
| Minhas principais dificuldades na remoção do code smell são:                   | Notei que o CATSPY também cataloga Sensitive Equality (comparar repr com string). Aqui o próprio `__repr__` é a unidade testada, então comparar a string é a verificação correta, e não um caso desse smell. |
| Usei o CATSPY para:                                                            | Definição, exemplo refatorado e para checar se a solução criaria Sensitive Equality.                                                                                                                         |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Substituição do print por asserção sobre o repr não vinculado (`CachedMethod(func=..., scoped=False, typed=False)`). Redundant Print zerado no arquivo.                                                      |

## R25 | Non-Functional Statement | tests/test_cacheutils.py::test_lru_basic (linha 120) | commit [c300b43](https://github.com/harielfr/boltons/commit/c300b43)

**Autor:** Matheus dos Santos · **CATSPY:** [Non-Functional Statement](https://cat-spy.vercel.app/smells/Non-Functional%20Statement)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                        |
| ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Non-Functional Statement: o `pass` dentro de `except KeyError`, num bloco `try/except/else: assert False` que verifica se `lru.pop('bye')` lança KeyError.                                                                                      |
| Minhas principais dificuldades na remoção do code smell são:                   | O `pass` não pode ser apagado sozinho, porque o `except` ficaria sem corpo. O smell só some quando a lógica de "esperar uma exceção" muda de forma. Isso também remove o Exception Handling da linha 117, que não é uma das minhas ocorrências. |
| Usei o CATSPY para:                                                            | Definição de Non-Functional Statement e a refatoração de Exception Handling (assertRaises), que é a estratégia que se aplica aqui.                                                                                                              |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Troca do bloco de 6 linhas por `with pytest.raises(KeyError): lru.pop('bye')`, equivalente pytest do assertRaises.                                                                                                                              |

## R26 | Non-Functional Statement | tests/test_cacheutils.py::test_lru_basic (linha 130) | commit [da77277](https://github.com/harielfr/boltons/commit/da77277)

**Autor:** Matheus dos Santos · **CATSPY:** [Non-Functional Statement](https://cat-spy.vercel.app/smells/Non-Functional%20Statement)

| Campo                                                                          | Resposta                                                                                          |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Non-Functional Statement: mesmo padrão do R04, agora para `lru.popitem()` em um LRU vazio.        |
| Minhas principais dificuldades na remoção do code smell são:                   | Nenhuma nova. Mantive um commit separado do R04 para cumprir a regra de um commit por ocorrência. |
| Usei o CATSPY para:                                                            | Definição e exemplo refatorado.                                                                   |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | `with pytest.raises(KeyError): lru.popitem()`. Remove também o Exception Handling da linha 127.   |

## R27 | Non-Functional Statement | tests/test_debugutils_trace.py::test_trace_exc (linha 43) | commit [891537d](https://github.com/harielfr/boltons/commit/891537d)

**Autor:** Matheus dos Santos · **CATSPY:** [Non-Functional Statement](https://cat-spy.vercel.app/smells/Non-Functional%20Statement)

| Campo                                                                          | Resposta                                                                                                                                                             |
| ------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Non-Functional Statement: o `pass` que forma o corpo de uma classe de exceção local (`class TestException(Exception): pass`).                                        |
| Minhas principais dificuldades na remoção do code smell são:                   | Diferente do exemplo do CATSPY, o `pass` não está num método vazio, mas no corpo de uma classe, onde algum comando é obrigatório. Não há asserção a acrescentar ali. |
| Usei o CATSPY para:                                                            | Definição. A estratégia (remover a instrução sem finalidade) precisou ser adaptada.                                                                                  |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Troca do `pass` por uma docstring que explica para que serve a exceção. É a forma idiomática de um corpo de classe vazio e documenta a intenção.                     |

## R28 | Non-Functional Statement | tests/test_dictutils.py::test_pop (linha 253) | commit [a65a643](https://github.com/harielfr/boltons/commit/a65a643)

**Autor:** Matheus dos Santos · **CATSPY:** [Non-Functional Statement](https://cat-spy.vercel.app/smells/Non-Functional%20Statement)

| Campo                                                                          | Resposta                                                                                                                                             |
| ------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Non-Functional Statement: `except KeyError: pass` depois de `omd.pop('odd'); assert False`.                                                          |
| Minhas principais dificuldades na remoção do code smell são:                   | O `assert False` fica dentro do `try`. Se algum dia virasse `assert` sem efeito (por exemplo, com `python -O`), o teste passaria sem verificar nada. |
| Usei o CATSPY para:                                                            | Definição e exemplo refatorado de Exception Handling.                                                                                                |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | `with pytest.raises(KeyError): omd.pop('odd')`. Remove também o Exception Handling da linha 249.                                                     |

## R29 | Non-Functional Statement | tests/test_dictutils.py::test_pop_all (linha 305) | commit [f2fb31d](https://github.com/harielfr/boltons/commit/f2fb31d)

**Autor:** Matheus dos Santos · **CATSPY:** [Non-Functional Statement](https://cat-spy.vercel.app/smells/Non-Functional%20Statement)

| Campo                                                                          | Resposta                                                                                            |
| ------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Non-Functional Statement: mesmo padrão do R07, para `omd.popall('odd')`.                            |
| Minhas principais dificuldades na remoção do code smell são:                   | Nenhuma nova. Suíte completa rodada após este commit: 525 aprovados.                                |
| Usei o CATSPY para:                                                            | Definição e exemplo refatorado.                                                                     |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | `with pytest.raises(KeyError): omd.popall('odd')`. Remove também o Exception Handling da linha 301. |

## R30 | Undefined Test | tests/test_listutils.py::\_multi_sublist_bl (linha 143) | commit [e1c3777](https://github.com/harielfr/boltons/commit/e1c3777)

**Autor:** Matheus dos Santos · **CATSPY:** [Undefined Test](https://cat-spy.vercel.app/smells/Undefined%20Test)

| Campo                                                                          | Resposta                                                                                                                                                                                                                            |
| ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Undefined Test: `_multi_sublist_bl` tem asserções, mas não começa com `test`. É uma fábrica que monta um BarrelList com várias sublistas e é usada por 4 testes.                                                                    |
| Minhas principais dificuldades na remoção do code smell são:                   | O CATSPY sugere renomear para `test_...`. Aqui isso faria o pytest coletar a fábrica como teste. Ela retorna um valor (aviso PytestReturnNotNoneWarning, e o smell Return in Test) e continuaria sendo chamada pelos outros testes. |
| Usei o CATSPY para:                                                            | Definição e as duas opções de refatoração (renomear ou remover). Usei uma combinação das duas.                                                                                                                                      |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Separei responsabilidades: `_multi_sublist_bl` virou só fábrica, sem asserções, e as asserções foram para um teste novo, `test_multi_sublist_bl_splits_and_keeps_order`, que o pytest coleta. 1 teste a mais (526).                 |

## R31 | Exception Handling | tests/test_ioutils.py::test_rollover_custom_directory (linha 44) | commit [cfa381d](https://github.com/harielfr/boltons/commit/cfa381d)

**Autor:** Matheus dos Santos · **CATSPY:** [Exception Handling](https://cat-spy.vercel.app/smells/Exception%20Handling)

| Campo                                                                          | Resposta                                                                                                                                                                                                                               |
| ------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Exception Handling: um `try/finally` cujo único papel era apagar, com `os.rmdir`, o diretório criado por `mkdtemp()`.                                                                                                                  |
| Minhas principais dificuldades na remoção do code smell são:                   | O exemplo do CATSPY trata de `try/except` para esperar exceções. Aqui o `try` é de limpeza de recurso, então assertRaises não se aplica. Também confirmei que o arquivo temporário é fechado antes da limpeza (importante no Windows). |
| Usei o CATSPY para:                                                            | Definição e efeitos (o try deixa a intenção do teste menos clara). A estratégia foi adaptada: usar o recurso da linguagem em vez de tratamento manual.                                                                                 |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | `with TemporaryDirectory() as custom_dir:`, que cria e apaga o diretório sozinho. O import de `mkdtemp` virou `TemporaryDirectory`. Testes do arquivo: 88 aprovados.                                                                   |

## R32 | Exception Handling | tests/test_ioutils.py::test_buffered_writer_flush (linha 306) | commit [8bf67d1](https://github.com/harielfr/boltons/commit/8bf67d1)

**Autor:** Matheus dos Santos · **CATSPY:** [Exception Handling](https://cat-spy.vercel.app/smells/Exception%20Handling)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                                                        |
| ------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Exception Handling: `try: writer.close() except BlockingIOError: stream.close()`, aninhado no `finally` do teste.                                                                                                                                                                                                                                               |
| Minhas principais dificuldades na remoção do code smell são:                   | Entender por que o tratamento existia. Li `SpooledBytesIO.write` em boltons/ioutils.py: desde o commit #513 ele retorna a contagem de bytes. O BlockingIOError só acontecia quando `write` retornava None, então o `except` escondia uma regressão em vez de proteger o teste. Fiz este try (o interno) antes do externo (R12) porque ele está dentro do outro. |
| Usei o CATSPY para:                                                            | Definição e efeitos negativos (o tratamento manual reduz a confiabilidade e esconde falhas).                                                                                                                                                                                                                                                                    |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Remoção do tratamento manual: `writer.close()` direto. Se o close voltar a falhar, o teste falha e mostra o problema.                                                                                                                                                                                                                                           |

## R33 | Exception Handling | tests/test_ioutils.py::test_buffered_writer_flush (linha 301) | commit [3ebbda6](https://github.com/harielfr/boltons/commit/3ebbda6)

**Autor:** Matheus dos Santos · **CATSPY:** [Exception Handling](https://cat-spy.vercel.app/smells/Exception%20Handling)

| Campo                                                                          | Resposta                                                                                                                                            |
| ------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Exception Handling: o `try/finally` externo, que só garantia o `writer.close()`.                                                                    |
| Minhas principais dificuldades na remoção do code smell são:                   | Garantir que o `BufferedWriter` continue sendo fechado mesmo se a asserção falhar, sem o `try`.                                                     |
| Usei o CATSPY para:                                                            | Definição. A estratégia foi adaptada, como no R10 (gerenciador de contexto em vez de assertRaises).                                                 |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | `with io.BufferedWriter(stream) as writer:` no lugar de `try/finally`. Testes de ioutils: 88 aprovados, 12 subtests. Suíte completa: 526 aprovados. |

## R34 | Unknown Test | tests/test_cacheutils.py::test_lri_cache_eviction (linha 55) | commit [f0e33cb](https://github.com/harielfr/boltons/commit/f0e33cb)

**Autor:** Matheus dos Santos · **CATSPY:** [Unknown Test](https://cat-spy.vercel.app/smells/Unknown%20Test)

| Campo                                                                          | Resposta                                                                                                                                                  |
| ------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Unknown Test: teste de regressão que só insere chaves num LRI(2) e passa se nada lançar exceção.                                                          |
| Minhas principais dificuldades na remoção do code smell são:                   | Descobrir o estado final esperado. Usei a docstring e os comentários do teste, que descrevem a fila de inserção passo a passo: ao inserir key3, key1 sai. |
| Usei o CATSPY para:                                                            | Definição e exemplo refatorado (adicionar a asserção que valida o resultado esperado).                                                                    |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Inclusão de `assert dict(test_cache) == {"key2": "value2", "key3": "value3"}`, com mensagem explicativa.                                                  |

## R35 | Unknown Test | tests/test_iterutils.py::test_unremappable (linha 87) | commit [7cc668c](https://github.com/harielfr/boltons/commit/7cc668c)

**Autor:** Matheus dos Santos · **CATSPY:** [Unknown Test](https://cat-spy.vercel.app/smells/Unknown%20Test)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                        |
| ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Unknown Test: o teste só tem `with pytest.raises(TypeError): remap(obj)`.                                                                                                                                                                       |
| Minhas principais dificuldades na remoção do code smell são:                   | O teste já verifica algo (a exceção), então é um caso de fronteira do TEMPY, que não conta `pytest.raises` como asserção. Busquei uma asserção que agregasse valor: qualquer TypeError fazia o teste passar, inclusive um vindo de outro ponto. |
| Usei o CATSPY para:                                                            | Definição (sem asserções explícitas fica difícil entender o objetivo do teste).                                                                                                                                                                 |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Captura da exceção com `as exc_info` e `assert 'expected remappable root' in str(exc_info.value)`.                                                                                                                                              |

## R36 | Unknown Test | tests/test_iterutils.py::test_noncallables (linha 104) | commit [4306458](https://github.com/harielfr/boltons/commit/4306458)

**Autor:** Matheus dos Santos · **CATSPY:** [Unknown Test](https://cat-spy.vercel.app/smells/Unknown%20Test)

| Campo                                                                          | Resposta                                                                                                                                 |
| ------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Unknown Test: três `pytest.raises(TypeError)` seguidos (visit, enter e exit não chamáveis), sem asserção.                                |
| Minhas principais dificuldades na remoção do code smell são:                   | Com três blocos iguais, uma falha não mostrava qual parâmetro estava errado. Rodei `remap` no terminal para ver a mensagem de cada caso. |
| Usei o CATSPY para:                                                            | Definição e exemplo refatorado.                                                                                                          |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Cada bloco captura sua exceção e afirma que a mensagem cita o parâmetro certo (`'visit expected callable'` e assim por diante).          |

## R37 | Unknown Test | tests/test_dictutils.py::test_frozendict_ior (linha 537) | commit [7ea9709](https://github.com/harielfr/boltons/commit/7ea9709)

**Autor:** Matheus dos Santos · **CATSPY:** [Unknown Test](https://cat-spy.vercel.app/smells/Unknown%20Test)

| Campo                                                                          | Resposta                                                                                                                                               |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------ |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Unknown Test: a atualização in-place (operador de união com atribuição, fd \\                                                                          | = fd) dentro de pytest.raises(TypeError, match=...), sem asserção. |
| Minhas principais dificuldades na remoção do code smell são:                   | A mensagem da exceção já era verificada pelo `match`. Faltava verificar a consequência que importa para um objeto imutável: o conteúdo não pode mudar. |
| Usei o CATSPY para:                                                            | Definição e exemplo refatorado.                                                                                                                        |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | `assert fd == data` depois do bloco, com mensagem. Suíte completa: 526 aprovados.                                                                      |

## R38 | Verbose Test | tests/test_dictutils.py::test_one_to_one (linhas 385 a 442) | commit [c624b10](https://github.com/harielfr/boltons/commit/c624b10)

**Autor:** Matheus dos Santos · **CATSPY:** [Verbose Test](https://cat-spy.vercel.app/smells/Verbose%20Test)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                                                                                                                                              |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Verbose Test: um teste de 58 linhas que verifica todas as operações de OneToOne em sequência, com uma closure `ck()` para as asserções.                                                                                                                                                                                                                                               |
| Minhas principais dificuldades na remoção do code smell são:                   | Cada passo dependia do estado deixado pelo anterior, então cada teste novo precisou montar seu próprio estado inicial. Na primeira tentativa usei uma função auxiliar no lugar do `ck()`. O TEMPY passou a acusar 1 Undefined Test e 6 Unknown Test, porque não enxerga asserções dentro de funções auxiliares. Descartei essa versão e deixei as asserções explícitas em cada teste. |
| Usei o CATSPY para:                                                            | Exemplo refatorado (dividir em métodos menores, cada um com uma responsabilidade).                                                                                                                                                                                                                                                                                                    |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Extração em 8 testes: setitem, delitem, copy, pop/popitem, setdefault, update, valores sobrepostos e unique. Todas as verificações originais foram mantidas, e test_one_to_one_unique passou a checar a mensagem do ValueError. Sem smells novos.                                                                                                                                     |

## R39 | Conditional Test Logic | tests/test_cacheutils.py::test_lri (linha 34 (if)) | commit [7ced183](https://github.com/harielfr/boltons/commit/7ced183)

**Autor:** Matheus dos Santos · **CATSPY:** [Conditional Test Logic](https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Conditional Test Logic: `if least_recent_insert_index >= 0:`, que pulava a verificação de remoção enquanto o cache ainda não estava cheio. Dentro dele havia um `for` que checava as chaves uma a uma.                                                  |
| Minhas principais dificuldades na remoção do code smell são:                   | test_lri tem complexidade ciclomática 12 (Radon). O `if` e o `for` externo (R19) estão aninhados, então resolvi o `if` primeiro, para cada commit ter uma ocorrência. Precisei evitar criar outro `if` ou `for` no lugar.                               |
| Usei o CATSPY para:                                                            | Definição e efeitos (a condicional fazia a fase inicial do cache ficar sem verificação).                                                                                                                                                                |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Uma asserção única que vale em todos os passos: `assert set(bc) == set(letters[max(0, idx - cache_size + 1):idx + 1])`. Ela é mais rigorosa que a original (detecta também chaves sobrando) e removeu o `for` interno (linha 37) como efeito colateral. |

## R40 | Conditional Test Logic | tests/test_cacheutils.py::test_lri (linha 30 (for)) | commit [175e86f](https://github.com/harielfr/boltons/commit/175e86f)

**Autor:** Matheus dos Santos · **CATSPY:** [Conditional Test Logic](https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic)

| Campo                                                                          | Resposta                                                                                                                                                                                                                                                            |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Conditional Test Logic: `for idx, char in enumerate(string.ascii_letters):`, um laço de 52 passos com asserções em cada passo.                                                                                                                                      |
| Minhas principais dificuldades na remoção do code smell são:                   | Preservar a verificação passo a passo sem o laço, e manter a segunda metade do teste (reinserção de chave), que depende do cache cheio.                                                                                                                             |
| Usei o CATSPY para:                                                            | Exemplo refatorado (extrair cada caso para um teste separado), aplicado com `pytest.mark.parametrize`.                                                                                                                                                              |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Novo teste `test_lri_on_miss_keeps_most_recent_inserts`, parametrizado de 1 a 52 consultas: cada passo do antigo laço virou um caso independente, e uma falha indica o passo. test_lri ficou só com o cenário de reinserção. CC de test_lri: 12 para 7. +52 testes. |

## R41 | Conditional Test Logic | tests/test_dictutils.py::test_clear (linha 129) | commit [2984e57](https://github.com/harielfr/boltons/commit/2984e57)

**Autor:** Matheus dos Santos · **CATSPY:** [Conditional Test Logic](https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic)

| Campo                                                                          | Resposta                                                                                                                |
| ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell:         | Conditional Test Logic: `for itemset in _ITEMSETS:` envolvendo todo o corpo do teste.                                   |
| Minhas principais dificuldades na remoção do code smell são:                   | Poucas: o laço só repetia o mesmo cenário para 5 entradas. Com o laço, uma falha não dizia qual entrada quebrou.        |
| Usei o CATSPY para:                                                            | Exemplo refatorado (um caso de teste por situação).                                                                     |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | `@pytest.mark.parametrize("itemset", _ITEMSETS)`: 5 casos independentes. Suíte completa final: 589 aprovados, 0 falhas. |

## Observações e dúvidas para o monitor

- Os registros de Matheus mantêm o texto do diário individual dele, só com os IDs renumerados (R01–R20 viraram R22–R41). As linhas citadas nas entradas dele são as do commit base.
- Quando um `pytest.raises` removeu juntos um Non-Functional Statement e um Exception Handling do mesmo bloco, contamos **uma** ocorrência (a da lista). Os smells vizinhos removidos junto não entraram na contagem das 40 e estão citados nas entradas. Perguntar ao monitor se isso está certo.
- R04 (Enzo) criou um Non-Functional Statement novo, e ele foi removido no commit seguinte da mesma região, R21 (`6aa6b6b`). Nas versões finais, nenhum tipo de smell aumentou (TEMPY: 207 para 157).
- Duas primeiras versões foram descartadas antes do commit por criarem smells novos: R17 de Enzo (2 Unknown Test) e R38 de Matheus (1 Undefined Test e 6 Unknown Test).
- Perguntar ao monitor se o diário deve ser um só da dupla ou um por integrante (os individuais continuam disponíveis em `artefatos/`).
