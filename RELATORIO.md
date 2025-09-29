# Relatório: Mini-Projeto 1 - Quebra-Senhas Paralelo

**Aluno(s):** Carlos Eduardo Diniz de Almeida 10444407,
Guilherme Silveira Giacomini (10435311), 
Lucas Distretti Claudio (10410094)
---

## 1. Estratégia de Paralelização


**Como você dividiu o espaço de busca entre os workers?**

Primeiramente foi necessário conseguir o espaço de busca total. A partir disso, foi necessário dividir esse espaço de busca total pela quantidade de workers, no qual, retornava o espaço de busca de cada worker.

**Código relevante:** Cole aqui a parte do coordinator.c onde você calcula a divisão:
```c
long long passwords_per_worker = total_space/num_workers;
long long remaining = total_space % num_workers;
```

---

## 2. Implementação das System Calls

**Descreva como você usou fork(), execl() e wait() no coordinator:**

Depois de ter calculado o intervalo de busca, dentro do for, foi necessário executar os fork(). O filho ia para a função exec(), e o pai armazenava o pid no vetor de workers. Por fim, o wait esperava todos os workers em que o pai armazenou no vetor de workers.

**Código do fork/exec:**
```c
for (int i = 0; i < num_workers; i++) {
        
        long long start_interval;
        long long end_interval;
        
        
        if (i == 0){
            start_interval = 0;
            end_interval = passwords_per_worker;
        }
        else if (i == num_workers -1){
            start_interval = (passwords_per_worker * i) + 1;
            end_interval = passwords_per_worker * (i+1) + remaining_per_worker + remaining_of_remaining;
        }
        else {
            start_interval = (passwords_per_worker * i) + 1;
            end_interval = passwords_per_worker * (i+1) + remaining_per_worker;
        }
        
        char start[password_len + 1];
        index_to_password(start_interval, charset, charset_len, password_len, start);
        start[password_len] = '\0';

        char end[password_len + 1];
        index_to_password(end_interval, charset, charset_len, password_len, end);
        end[password_len] = '\0';

        
        pid_t pid = fork();
        
        
        if (pid < 0){
            fprintf(stderr, "Erro: falha no fork.\n");
            return 1;
        }
        else if(pid == 0){
            char len_str[8], id_str[8];
            snprintf(len_str, sizeof(len_str), "%d", password_len);
            snprintf(id_str, sizeof(id_str), "%d", i + 1);

            execl("./worker", "worker", target_hash, start, end, charset, len_str, id_str, (char*)NULL);

            perror("Error no execl");   
            exit(1);
        } 
        else {
            workers[i] = pid;
        }
        
    }
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

O resto não estava distribuído entre os workers. Para resolver isso, dividimos o resto pela quantidade de workers, que retornava o valor restante que cada worker iria ficar. E caso, ainda assim, restasse, o último worker ficaria responsável com essa parte.

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
- [x] Relatório preenchido