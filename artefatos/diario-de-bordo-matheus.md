# Diário de Bordo – Matheus dos Santos (boltons)

Nov 1, 2026 · @Matheus

## Test Smells em projetos OpenSource

**Disciplina:** Qualidade de Software (UFC, Profa. Carla Ilane Moreira Bezerra)

**Equipe:** Enzo Hariel e Matheus dos Santos. Este diário cobre as 20 refatorações de Matheus.

**Projeto:** boltons ([github.com/mahmoud/boltons](https://github.com/mahmoud/boltons))

**Fork:** [github.com/harielfr/boltons](https://github.com/harielfr/boltons), branch [refatoracao-test-smells-matheus](https://github.com/harielfr/boltons/tree/refatoracao-test-smells-matheus)

**Commit base:** 4e5faa3d7e4008d89e0d8bf1ea87b6d9a061a16d

**Arquivos sob responsabilidade de Matheus:** test\_cacheutils.py, test\_dictutils.py, test\_iterutils.py, test\_listutils.py, test\_ioutils.py, test\_debugutils\_trace.py

## Resumo

Foram refatoradas as 20 ocorrências previstas, de 7 tipos, com um commit por ocorrência e todos os testes passando em cada commit. Nenhum smell novo foi introduzido. Os números abaixo são de conferência local: as métricas oficiais "depois" serão geradas pelo Enzo após o merge.

| Medição | Commit base | Após as 20 refatorações |
| --- | --- | --- |
| Testes aprovados (pytest) | 525 (0 falhas) | 589 (0 falhas) |
| Test smells no projeto (TEMPY, modo conferência) | 207 | 182 |
| Smells nos 6 arquivos de Matheus | 108 | 83 |
| Cobertura de linhas e branches do boltons | 68% | 68% |

Os 64 testes a mais vêm de parametrizações e divisões: 52 casos em test\_lri\_on\_miss\_keeps\_most\_recent\_inserts, 5 casos em test\_clear (antes 1), 8 testes no lugar de test\_one\_to\_one e 1 teste novo em test\_listutils.py. Nenhum teste existente foi removido sem que sua verificação fosse mantida.

A queda de 25 smells inclui as 20 ocorrências-alvo e 5 removidas como efeito colateral: 4 Exception Handling (test\_lru\_basic, test\_pop, test\_pop\_all) e 1 Conditional Test Logic (for interno de test\_lri).

| Tipo | Ocorrências | IDs |
| --- | --- | --- |
| Redundant Print | 3 | R01 a R03 |
| Non-Functional Statement | 5 | R04 a R08 |
| Undefined Test | 1 | R09 |
| Exception Handling | 3 | R10 a R12 |
| Unknown Test | 4 | R13 a R16 |
| Verbose Test | 1 | R17 |
| Conditional Test Logic | 3 | R18 a R20 |

## Registros das refatorações

Os registros seguem a ordem dos commits. As linhas indicadas são as do commit base, segundo o TEMPY. Em cada um, os testes do arquivo e a conferência do TEMPY (`extract_smells_tempy.py --arquivo`) foram rodados antes do commit. A cada 3 ou 4 refatorações, rodei também a suíte completa.

### R01 – Redundant Print

**Localização:** tests/test\_cacheutils.py, test\_callable\_cached\_dec, linha 297 · **Commit:** dac7ea0 · **CATSPY:** [Redundant Print](https://cat-spy.vercel.app/smells/Redundant%20Print)

| Pergunta do modelo | Resposta |
| --- | --- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell: | Redundant Print: `print(repr(func))` no fim do teste, sem nenhuma verificação sobre o valor impresso. |
| Minhas principais dificuldades na remoção do code smell são: | Decidir entre apagar o print ou transformá-lo em asserção. O print existia para exercitar o `__repr__` de CachedFunction, então apagar perderia essa intenção. Li `CachedFunction.__repr__` em boltons/cacheutils.py para saber o formato exato. |
| Usei o CATSPY para: | Definição e exemplo refatorado (substituir o print por uma asserção com a informação relevante). |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Substituição do print por asserção: `assert repr(func) == f'CachedFunction(func={inner_func!r})'`. Testes do arquivo: 23 aprovados. |

### R02 – Redundant Print

**Localização:** tests/test\_cacheutils.py, test\_cachedmethod, linha 356 · **Commit:** 19fbfde · **CATSPY:** [Redundant Print](https://cat-spy.vercel.app/smells/Redundant%20Print)

| Pergunta do modelo | Resposta |
| --- | --- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell: | Redundant Print: `print(repr(car_two.door))`, logo abaixo do comentário "always check the repr". |
| Minhas principais dificuldades na remoção do code smell são: | O método está vinculado a uma instância, e o repr muda nesse caso (inclui `bound_to`). Foi preciso obter a função original por `Car.door.func` para montar o valor esperado. |
| Usei o CATSPY para: | Definição e exemplo refatorado. |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Substituição do print por asserção sobre o repr do CachedMethod vinculado (`<CachedMethod func=... scoped=False typed=False bound_to=...>`). Um commit só para este print. |

### R03 – Redundant Print

**Localização:** tests/test\_cacheutils.py, test\_cachedmethod, linha 357 · **Commit:** b9c420e · **CATSPY:** [Redundant Print](https://cat-spy.vercel.app/smells/Redundant%20Print)

| Pergunta do modelo | Resposta |
| --- | --- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell: | Redundant Print: `print(repr(Car.door))`, o método acessado pela classe (não vinculado). |
| Minhas principais dificuldades na remoção do code smell são: | Notei que o CATSPY também cataloga Sensitive Equality (comparar repr com string). Aqui o próprio `__repr__` é a unidade testada, então comparar a string é a verificação correta, e não um caso desse smell. |
| Usei o CATSPY para: | Definição, exemplo refatorado e para checar se a solução criaria Sensitive Equality. |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Substituição do print por asserção sobre o repr não vinculado (`CachedMethod(func=..., scoped=False, typed=False)`). Redundant Print zerado no arquivo. |

### R04 – Non-Functional Statement

**Localização:** tests/test\_cacheutils.py, test\_lru\_basic, linha 120 · **Commit:** c300b43 · **CATSPY:** [Non-Functional Statement](https://cat-spy.vercel.app/smells/Non-Functional%20Statement), [Exception Handling](https://cat-spy.vercel.app/smells/Exception%20Handling)

| Pergunta do modelo | Resposta |
| --- | --- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell: | Non-Functional Statement: o `pass` dentro de `except KeyError`, num bloco `try/except/else: assert False` que verifica se `lru.pop('bye')` lança KeyError. |
| Minhas principais dificuldades na remoção do code smell são: | O `pass` não pode ser apagado sozinho, porque o `except` ficaria sem corpo. O smell só some quando a lógica de "esperar uma exceção" muda de forma. Isso também remove o Exception Handling da linha 117, que não é uma das minhas ocorrências. |
| Usei o CATSPY para: | Definição de Non-Functional Statement e a refatoração de Exception Handling (assertRaises), que é a estratégia que se aplica aqui. |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Troca do bloco de 6 linhas por `with pytest.raises(KeyError): lru.pop('bye')`, equivalente pytest do assertRaises. |

### R05 – Non-Functional Statement

**Localização:** tests/test\_cacheutils.py, test\_lru\_basic, linha 130 · **Commit:** da77277 · **CATSPY:** [Non-Functional Statement](https://cat-spy.vercel.app/smells/Non-Functional%20Statement)

| Pergunta do modelo | Resposta |
| --- | --- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell: | Non-Functional Statement: mesmo padrão do R04, agora para `lru.popitem()` em um LRU vazio. |
| Minhas principais dificuldades na remoção do code smell são: | Nenhuma nova. Mantive um commit separado do R04 para cumprir a regra de um commit por ocorrência. |
| Usei o CATSPY para: | Definição e exemplo refatorado. |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | `with pytest.raises(KeyError): lru.popitem()`. Remove também o Exception Handling da linha 127. |

### R06 – Non-Functional Statement

**Localização:** tests/test\_debugutils\_trace.py, test\_trace\_exc, linha 43 · **Commit:** 891537d · **CATSPY:** [Non-Functional Statement](https://cat-spy.vercel.app/smells/Non-Functional%20Statement)

| Pergunta do modelo | Resposta |
| --- | --- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell: | Non-Functional Statement: o `pass` que forma o corpo de uma classe de exceção local (`class TestException(Exception): pass`). |
| Minhas principais dificuldades na remoção do code smell são: | Diferente do exemplo do CATSPY, o `pass` não está num método vazio, mas no corpo de uma classe, onde algum comando é obrigatório. Não há asserção a acrescentar ali. |
| Usei o CATSPY para: | Definição. A estratégia (remover a instrução sem finalidade) precisou ser adaptada. |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Troca do `pass` por uma docstring que explica para que serve a exceção. É a forma idiomática de um corpo de classe vazio e documenta a intenção. |

### R07 – Non-Functional Statement

**Localização:** tests/test\_dictutils.py, test\_pop, linha 253 · **Commit:** a65a643 · **CATSPY:** [Non-Functional Statement](https://cat-spy.vercel.app/smells/Non-Functional%20Statement)

| Pergunta do modelo | Resposta |
| --- | --- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell: | Non-Functional Statement: `except KeyError: pass` depois de `omd.pop('odd'); assert False`. |
| Minhas principais dificuldades na remoção do code smell são: | O `assert False` fica dentro do `try`. Se algum dia virasse `assert` sem efeito (por exemplo, com `python -O`), o teste passaria sem verificar nada. |
| Usei o CATSPY para: | Definição e exemplo refatorado de Exception Handling. |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | `with pytest.raises(KeyError): omd.pop('odd')`. Remove também o Exception Handling da linha 249. |

### R08 – Non-Functional Statement

**Localização:** tests/test\_dictutils.py, test\_pop\_all, linha 305 · **Commit:** f2fb31d · **CATSPY:** [Non-Functional Statement](https://cat-spy.vercel.app/smells/Non-Functional%20Statement)

| Pergunta do modelo | Resposta |
| --- | --- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell: | Non-Functional Statement: mesmo padrão do R07, para `omd.popall('odd')`. |
| Minhas principais dificuldades na remoção do code smell são: | Nenhuma nova. Suíte completa rodada após este commit: 525 aprovados. |
| Usei o CATSPY para: | Definição e exemplo refatorado. |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | `with pytest.raises(KeyError): omd.popall('odd')`. Remove também o Exception Handling da linha 301. |

### R09 – Undefined Test

**Localização:** tests/test\_listutils.py, \_multi\_sublist\_bl, linha 143 · **Commit:** e1c3777 · **CATSPY:** [Undefined Test](https://cat-spy.vercel.app/smells/Undefined%20Test)

| Pergunta do modelo | Resposta |
| --- | --- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell: | Undefined Test: `_multi_sublist_bl` tem asserções, mas não começa com `test`. É uma fábrica que monta um BarrelList com várias sublistas e é usada por 4 testes. |
| Minhas principais dificuldades na remoção do code smell são: | O CATSPY sugere renomear para `test_...`. Aqui isso faria o pytest coletar a fábrica como teste. Ela retorna um valor (aviso PytestReturnNotNoneWarning, e o smell Return in Test) e continuaria sendo chamada pelos outros testes. |
| Usei o CATSPY para: | Definição e as duas opções de refatoração (renomear ou remover). Usei uma combinação das duas. |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Separei responsabilidades: `_multi_sublist_bl` virou só fábrica, sem asserções, e as asserções foram para um teste novo, `test_multi_sublist_bl_splits_and_keeps_order`, que o pytest coleta. 1 teste a mais (526). |

### R10 – Exception Handling

**Localização:** tests/test\_ioutils.py, test\_rollover\_custom\_directory, linha 44 · **Commit:** cfa381d · **CATSPY:** [Exception Handling](https://cat-spy.vercel.app/smells/Exception%20Handling)

| Pergunta do modelo | Resposta |
| --- | --- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell: | Exception Handling: um `try/finally` cujo único papel era apagar, com `os.rmdir`, o diretório criado por `mkdtemp()`. |
| Minhas principais dificuldades na remoção do code smell são: | O exemplo do CATSPY trata de `try/except` para esperar exceções. Aqui o `try` é de limpeza de recurso, então assertRaises não se aplica. Também confirmei que o arquivo temporário é fechado antes da limpeza (importante no Windows). |
| Usei o CATSPY para: | Definição e efeitos (o try deixa a intenção do teste menos clara). A estratégia foi adaptada: usar o recurso da linguagem em vez de tratamento manual. |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | `with TemporaryDirectory() as custom_dir:`, que cria e apaga o diretório sozinho. O import de `mkdtemp` virou `TemporaryDirectory`. Testes do arquivo: 88 aprovados. |

### R11 – Exception Handling

**Localização:** tests/test\_ioutils.py, test\_buffered\_writer\_flush, linha 306 · **Commit:** 8bf67d1 · **CATSPY:** [Exception Handling](https://cat-spy.vercel.app/smells/Exception%20Handling)

| Pergunta do modelo | Resposta |
| --- | --- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell: | Exception Handling: `try: writer.close() except BlockingIOError: stream.close()`, aninhado no `finally` do teste. |
| Minhas principais dificuldades na remoção do code smell são: | Entender por que o tratamento existia. Li `SpooledBytesIO.write` em boltons/ioutils.py: desde o commit #513 ele retorna a contagem de bytes. O BlockingIOError só acontecia quando `write` retornava None, então o `except` escondia uma regressão em vez de proteger o teste. Fiz este try (o interno) antes do externo (R12) porque ele está dentro do outro. |
| Usei o CATSPY para: | Definição e efeitos negativos (o tratamento manual reduz a confiabilidade e esconde falhas). |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Remoção do tratamento manual: `writer.close()` direto. Se o close voltar a falhar, o teste falha e mostra o problema. |

### R12 – Exception Handling

**Localização:** tests/test\_ioutils.py, test\_buffered\_writer\_flush, linha 301 · **Commit:** 3ebbda6 · **CATSPY:** [Exception Handling](https://cat-spy.vercel.app/smells/Exception%20Handling)

| Pergunta do modelo | Resposta |
| --- | --- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell: | Exception Handling: o `try/finally` externo, que só garantia o `writer.close()`. |
| Minhas principais dificuldades na remoção do code smell são: | Garantir que o `BufferedWriter` continue sendo fechado mesmo se a asserção falhar, sem o `try`. |
| Usei o CATSPY para: | Definição. A estratégia foi adaptada, como no R10 (gerenciador de contexto em vez de assertRaises). |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | `with io.BufferedWriter(stream) as writer:` no lugar de `try/finally`. Testes de ioutils: 88 aprovados, 12 subtests. Suíte completa: 526 aprovados. |

### R13 – Unknown Test

**Localização:** tests/test\_cacheutils.py, test\_lri\_cache\_eviction, linha 55 · **Commit:** f0e33cb · **CATSPY:** [Unknown Test](https://cat-spy.vercel.app/smells/Unknown%20Test)

| Pergunta do modelo | Resposta |
| --- | --- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell: | Unknown Test: teste de regressão que só insere chaves num LRI(2) e passa se nada lançar exceção. |
| Minhas principais dificuldades na remoção do code smell são: | Descobrir o estado final esperado. Usei a docstring e os comentários do teste, que descrevem a fila de inserção passo a passo: ao inserir key3, key1 sai. |
| Usei o CATSPY para: | Definição e exemplo refatorado (adicionar a asserção que valida o resultado esperado). |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Inclusão de `assert dict(test_cache) == {"key2": "value2", "key3": "value3"}`, com mensagem explicativa. |

### R14 – Unknown Test

**Localização:** tests/test\_iterutils.py, test\_unremappable, linha 87 · **Commit:** 7cc668c · **CATSPY:** [Unknown Test](https://cat-spy.vercel.app/smells/Unknown%20Test)

| Pergunta do modelo | Resposta |
| --- | --- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell: | Unknown Test: o teste só tem `with pytest.raises(TypeError): remap(obj)`. |
| Minhas principais dificuldades na remoção do code smell são: | O teste já verifica algo (a exceção), então é um caso de fronteira do TEMPY, que não conta `pytest.raises` como asserção. Busquei uma asserção que agregasse valor: qualquer TypeError fazia o teste passar, inclusive um vindo de outro ponto. |
| Usei o CATSPY para: | Definição (sem asserções explícitas fica difícil entender o objetivo do teste). |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Captura da exceção com `as exc_info` e `assert 'expected remappable root' in str(exc_info.value)`. |

### R15 – Unknown Test

**Localização:** tests/test\_iterutils.py, test\_noncallables, linha 104 · **Commit:** 4306458 · **CATSPY:** [Unknown Test](https://cat-spy.vercel.app/smells/Unknown%20Test)

| Pergunta do modelo | Resposta |
| --- | --- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell: | Unknown Test: três `pytest.raises(TypeError)` seguidos (visit, enter e exit não chamáveis), sem asserção. |
| Minhas principais dificuldades na remoção do code smell são: | Com três blocos iguais, uma falha não mostrava qual parâmetro estava errado. Rodei `remap` no terminal para ver a mensagem de cada caso. |
| Usei o CATSPY para: | Definição e exemplo refatorado. |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Cada bloco captura sua exceção e afirma que a mensagem cita o parâmetro certo (`'visit expected callable'` e assim por diante). |

### R16 – Unknown Test

**Localização:** tests/test\_dictutils.py, test\_frozendict\_ior, linha 537 · **Commit:** 7ea9709 · **CATSPY:** [Unknown Test](https://cat-spy.vercel.app/smells/Unknown%20Test)

| Pergunta do modelo | Resposta |
| --- | --- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell: | Unknown Test: a atualização in-place (operador de união com atribuição, fd \|= fd) dentro de pytest.raises(TypeError, match=...), sem asserção. |
| Minhas principais dificuldades na remoção do code smell são: | A mensagem da exceção já era verificada pelo `match`. Faltava verificar a consequência que importa para um objeto imutável: o conteúdo não pode mudar. |
| Usei o CATSPY para: | Definição e exemplo refatorado. |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | `assert fd == data` depois do bloco, com mensagem. Suíte completa: 526 aprovados. |

### R17 – Verbose Test

**Localização:** tests/test\_dictutils.py, test\_one\_to\_one, linhas 385 a 442 · **Commit:** c624b10 · **CATSPY:** [Verbose Test](https://cat-spy.vercel.app/smells/Verbose%20Test)

| Pergunta do modelo | Resposta |
| --- | --- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell: | Verbose Test: um teste de 58 linhas que verifica todas as operações de OneToOne em sequência, com uma closure `ck()` para as asserções. |
| Minhas principais dificuldades na remoção do code smell são: | Cada passo dependia do estado deixado pelo anterior, então cada teste novo precisou montar seu próprio estado inicial. Na primeira tentativa usei uma função auxiliar no lugar do `ck()`. O TEMPY passou a acusar 1 Undefined Test e 6 Unknown Test, porque não enxerga asserções dentro de funções auxiliares. Descartei essa versão e deixei as asserções explícitas em cada teste. |
| Usei o CATSPY para: | Exemplo refatorado (dividir em métodos menores, cada um com uma responsabilidade). |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Extração em 8 testes: setitem, delitem, copy, pop/popitem, setdefault, update, valores sobrepostos e unique. Todas as verificações originais foram mantidas, e test\_one\_to\_one\_unique passou a checar a mensagem do ValueError. Sem smells novos. |

### R18 – Conditional Test Logic

**Localização:** tests/test\_cacheutils.py, test\_lri, linha 34 (if) · **Commit:** 7ced183 · **CATSPY:** [Conditional Test Logic](https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic)

| Pergunta do modelo | Resposta |
| --- | --- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell: | Conditional Test Logic: `if least_recent_insert_index >= 0:`, que pulava a verificação de remoção enquanto o cache ainda não estava cheio. Dentro dele havia um `for` que checava as chaves uma a uma. |
| Minhas principais dificuldades na remoção do code smell são: | test\_lri tem complexidade ciclomática 12 (Radon). O `if` e o `for` externo (R19) estão aninhados, então resolvi o `if` primeiro, para cada commit ter uma ocorrência. Precisei evitar criar outro `if` ou `for` no lugar. |
| Usei o CATSPY para: | Definição e efeitos (a condicional fazia a fase inicial do cache ficar sem verificação). |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Uma asserção única que vale em todos os passos: `assert set(bc) == set(letters[max(0, idx - cache_size + 1):idx + 1])`. Ela é mais rigorosa que a original (detecta também chaves sobrando) e removeu o `for` interno (linha 37) como efeito colateral. |

### R19 – Conditional Test Logic

**Localização:** tests/test\_cacheutils.py, test\_lri, linha 30 (for) · **Commit:** 175e86f · **CATSPY:** [Conditional Test Logic](https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic)

| Pergunta do modelo | Resposta |
| --- | --- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell: | Conditional Test Logic: `for idx, char in enumerate(string.ascii_letters):`, um laço de 52 passos com asserções em cada passo. |
| Minhas principais dificuldades na remoção do code smell são: | Preservar a verificação passo a passo sem o laço, e manter a segunda metade do teste (reinserção de chave), que depende do cache cheio. |
| Usei o CATSPY para: | Exemplo refatorado (extrair cada caso para um teste separado), aplicado com `pytest.mark.parametrize`. |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | Novo teste `test_lri_on_miss_keeps_most_recent_inserts`, parametrizado de 1 a 52 consultas: cada passo do antigo laço virou um caso independente, e uma falha indica o passo. test\_lri ficou só com o cenário de reinserção. CC de test\_lri: 12 para 7. +52 testes. |

### R20 – Conditional Test Logic

**Localização:** tests/test\_dictutils.py, test\_clear, linha 129 · **Commit:** 2984e57 · **CATSPY:** [Conditional Test Logic](https://cat-spy.vercel.app/smells/Conditional%20Test%20Logic)

| Pergunta do modelo | Resposta |
| --- | --- |
| Eu estou atualmente trabalhando na refatoração do seguinte code smell: | Conditional Test Logic: `for itemset in _ITEMSETS:` envolvendo todo o corpo do teste. |
| Minhas principais dificuldades na remoção do code smell são: | Poucas: o laço só repetia o mesmo cenário para 5 entradas. Com o laço, uma falha não dizia qual entrada quebrou. |
| Usei o CATSPY para: | Exemplo refatorado (um caso de teste por situação). |
| Eu estou usando os seguintes métodos de refatoração para remover o code smell: | `@pytest.mark.parametrize("itemset", _ITEMSETS)`: 5 casos independentes. Suíte completa final: 589 aprovados, 0 falhas. |

## Observações para a dupla e dúvidas para o monitor

- **Ordem diferente do guia:** R11/R12 correspondem aos itens 12 e 11 do guia, e R18/R19 aos itens 19 e 18. Nos dois casos o trecho interno foi refatorado antes do externo que o envolve, para manter um commit por ocorrência.
- **Reservas:** nenhuma ocorrência de reserva foi usada.
- **Smells removidos de tabela:** 5 (4 Exception Handling e 1 Conditional Test Logic). Não estão contados nas 20.
- **Smells novos:** nenhum, conferido com o TEMPY por arquivo após cada commit.
- **Branch:** [refatoracao-test-smells-matheus](https://github.com/harielfr/boltons/tree/refatoracao-test-smells-matheus) no fork, com 20 commits sobre o commit base. Só os 6 arquivos de teste de Matheus foram alterados, então o merge com o Enzo não deve ter conflitos.
- **Para o PR (Enzo):** o upstream mahmoud/boltons avançou 28 commits além do commit base e alterou alguns desses arquivos de teste. Antes de abrir o PR, é preciso fazer rebase sobre o master do upstream. Testei em uma branch temporária: os 20 commits de Matheus entram sem conflito, e a suíte dá 684 aprovados, 1 ignorado e 0 falhas. As métricas oficiais continuam medidas sobre a branch baseada no commit da planilha.

* [ ] Perguntar ao monitor se, quando um `pytest.raises` remove juntos um Non-Functional Statement e um Exception Handling, conta uma ou duas ocorrências. Aqui contei uma.
* [ ] Perguntar ao monitor se o diário deve ser um documento só da dupla ou um por integrante.
* [ ] Preencher o formulário de triagem e, quando sair o link, o pós-teste (Matheus).
