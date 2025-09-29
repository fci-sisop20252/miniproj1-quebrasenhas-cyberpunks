# Relatório: Mini-Projeto 1 - Quebra-Senhas Paralelo

**Aluno(s):** Carlos Eduardo Diniz de Almeida 10444407,
Guilherme Silveira Giacomini (10435311), 
Lucas Distretti Claudio (10410094)
---

## 1. Estratégia de Paralelização


**Como você dividiu o espaço de busca entre os workers?**

[Explique seu algoritmo de divisão]

**Código relevante:** Cole aqui a parte do coordinator.c onde você calcula a divisão:
```c
// Cole seu código de divisão aqui
```

---

## 2. Implementação das System Calls

**Descreva como você usou fork(), execl() e wait() no coordinator:**

[Explique em um parágrafo como você criou os processos, passou argumentos e esperou pela conclusão]

**Código do fork/exec:**
```c
// Cole aqui seu loop de criação de workers
```

---

## 3. Comunicação Entre Processos

**Como você garantiu que apenas um worker escrevesse o resultado?**

Utilizando a flag `O_EXCL` na syscall open(). Ela garante que ocorra um erro caso um worker tente criar um arquivo "password_found.txt" que já exista. Isso garante que apenas um worker escreva o resultado.

`int fd = open(RESULT_FILE, O_CREAT | O_EXCL | O_WRONLY, 0644)`

**Como o coordinator consegue ler o resultado?**

Primeiro, ele verifica se o arquivo "password_found.txt" existe. Se não existir, significa que a senha não foi encontrada. Depois, ele lê o arquivo e o coloca num buffer. Dentro do buffer, o coordinator procura pelo sinal de dois pontos. Tudo aquilo que está após os dois pontos é a senha encontrada.

---

## 4. Análise de Performance
Complete a tabela com tempos reais de execução:
O speedup é o tempo do teste com 1 worker dividido pelo tempo com 4 workers.

| Teste | 1 Worker | 2 Workers | 4 Workers | Speedup (4w) |
|-------|----------|-----------|-----------|--------------|
| Hash: 202cb962ac59075b964b07152d234b70<br>Charset: "0123456789"<br>Tamanho: 3<br>Senha: "123" | 0.005s | 0.006s | 0.006s | 0.833 |
| Hash: 5d41402abc4b2a76b9719d911017c592<br>Charset: "abcdefghijklmnopqrstuvwxyz"<br>Tamanho: 5<br>Senha: "hello" | 6. 5.618s | 5.782s | 0.763s | 7.363 |

**O speedup foi linear? Por quê?**

Não, o speedup não foi linear. No primeiro caso mais curto, o overhead de criação dos processos foi maior que o da busca pela senha. Logo, com 4 workers, o tempo foi maior do que com 1. No teste 2, o speedup foi maior do que 4x. Isso ocorreu pois a senha foi encontrada rapidamente por um dos workers. Como os workers param após a senha ser encontrada, o programa foi rapidamente finalizado.

---

## 5. Desafios e Aprendizados
**Qual foi o maior desafio técnico que você enfrentou?**
[Descreva um problema e como resolveu. Ex: "Tive dificuldade com o incremento de senha, mas resolvi tratando-o como um contador em base variável"]

---

## Comandos de Teste Utilizados

```bash
# Teste básico
./coordinator "900150983cd24fb0d6963f7d28e17f72" 3 "abc" 2

# Teste de performance
time ./coordinator "202cb962ac59075b964b07152d234b70" 3 "0123456789" 1
time ./coordinator "202cb962ac59075b964b07152d234b70" 3 "0123456789" 4

# Teste com senha maior
time ./coordinator "5d41402abc4b2a76b9719d911017c592" 5 "abcdefghijklmnopqrstuvwxyz" 4
```
---

**Checklist de Entrega:**
- [x] Código compila sem erros
- [x] Todos os TODOs foram implementados
- [x] Testes passam no `./tests/simple_test.sh`
- [ ] Relatório preenchido