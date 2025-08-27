# 📝 Relatório do Laboratório 2 - Chamadas de Sistema

---

## 1️⃣ Exercício 1a - Observação printf() vs 1b - write()

### 💻 Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### 🔍 Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: 8 syscalls
- ex1b_write: 7 syscalls

**2. Por que há diferença entre os dois métodos? Consulte o docs/printf_vs_write.md**

```
O printf() é uma função da biblioteca padrão do C, que utiliza a syscall write() na usa implementação. 
A syscall write() apenas escreve uma quantidade definida de bytes definidos. 
A função printf() serve como uma camada de abstração de usa a syscall quando necessária. 
Dessa forma, permite a formatação de texto, manipulando dados além da saída no terminal. 
Além disso, nem sempre o número de syscalls write() utilizadas pelo programa é o mesmo dos usos da função printf().
```

**3. Qual método é mais previsível? Por quê você acha isso?**

```
Utilizar a chamada de sistema write() é mais previsível, pois cada chamada de write() no código significa uma syscall sendo realizada. Ao utilizar a função printf(), nem sempre o número de syscalls será o mesmo de chamadas da função. 
```

---

## 2️⃣ Exercício 2 - Leitura de Arquivo

### 📊 Resultados da execução:
- File descriptor: 3
- Bytes lidos: 127

### 🔧 Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### 🔍 Análise

**1. Qual file descriptor foi usado? Por que não começou em 0, 1 ou 2?**

```
Foi utilizado o file descriptor 3. Não foi utilizado 0, 1 ou 2 pois já são file descriptors reservados para a entrada e saída padrão do sistema. 
No caso, 0 é a entrada padrão (teclado), 1 é a saída padrão (terminal) e 2 é a saída de erro padrão (terminal).
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
A frase "Última linha do arquivo" foi exibida normalmente no terminal. 
Isso indica que o arquivo foi lido completamente e armazenado no buffer.
```

**3. Por que verificar retorno de cada syscall?**

```
Para saber informações importantes como o número do file descriptor e a quantidade de bytes lidos ou poder tratar os casos de erro adequadamente.
```

---

## 3️⃣ Exercício 3 - Contador com Loop

### 📋 Resultados (BUFFER_SIZE = 64):
- Linhas: 25 (esperado: 25)
- Caracteres: 1300
- Chamadas read(): 21
- Tempo: 0.000115 segundos

### 🧪 Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |      82           |      0.000184     |
| 64          |       21         |     0.000115      |
| 256         |        6       |     0.000069      |
| 1024        |        2      |      0.000059     |

### 🔍 Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
Quanto maior o buffer, maior a quantidade de dados que uma leitura consegue fazer. 
Logo, são necessárias menos chamadas da syscall read() na execução do programa.
```

**2. Todas as chamadas read() retornaram BUFFER_SIZE bytes? Discorra brevemente sobre**

```
Não. A última leitura não retorna BUFFER_SIZE bytes pois há uma quantidade menor de bytes para serem lidos do que o tamanho do buffer
```

**3. Qual é a relação entre syscalls e performance?**

```
Quanto menor o número de syscalls, mais rápida é a execução do programa. 
Isso ocorre pois uma chamada de sistema causa uma interrupção do kernel, sendo um processo custoso.
```

---

## 4️⃣ Exercício 4 - Cópia de Arquivo

### 📈 Resultados:
- Bytes copiados: 1364
- Operações: 6
- Tempo: 0.000164 segundos
- Throughput: 8122.14 KB/s

### ✅ Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [x] Idênticos [ ] Diferentes

### 🔍 Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
Para garantir que a cópia é exata e ocorreu sem erros.
```

**2. Que flags são essenciais no open() do destino?**

```
Para garantir que o programa tem permissão para escrever os bytes no destino.
```

**3. O número de reads e writes é igual? Por quê?**

```
Não. Existe um read a mais. É o último read, que lê 0 bytes, indicando que não há mais bytes para serem copiados da origem. 
```

**4. Como você saberia se o disco ficou cheio?**

```
Caso a operação de escrita write() tenha alguma falha, retornando -1 ou um número menor de bytes escritos em relação aos lidos.
```

**5. O que acontece se esquecer de fechar os arquivos?**

```
O file descriptor continua alocado até a finalização do programa, ocupando espaço desnecessário.
```

---

## 🎯 Análise Geral

### 📖 Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
As syscalls realizam tarefas que são permitidas somente ao kernel. 
Dessa forma, ocorre uma interrupção na execução iniciada pelo usuário e o kernel assume as funções desejadas.
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
Os file descriptors são importantes para o kernel saber os locais de escrita e leitura, sem acessar memória em locais não previstos. 
Ele conecta o kernel aos recursos de entrada e saída.
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
Quanto maior o buffer, menor é o número de syscalls necessárias para a completude da execução. 
Como a chamada de sistema é um processo custoso, economizar no número de chamadas resulta em economizar tempo.
```

### ⚡ Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** ex4_copia

**Por que você acha que foi mais rápido?**

```
Provavelmente porque o código do exercício é muito mais simples e menor do que o cp do sistema Linux 
e acaba sendo executado 1 ou 2 ms mais rápido em arquivos muito pequenos esse. Como o cp deve ser extremamente otimizado, 
a performance dele em arquivos maiores deve ser consideravelmente maior se comparado ao meu código.
```

---

## 📤 Entrega
Certifique-se de ter:
- [x] Todos os códigos com TODOs completados
- [x] Traces salvos em `traces/`
- [x] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```
# Bom trabalho!
