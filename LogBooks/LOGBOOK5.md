# Logbook 5 - Buffer Overflow

## Task 1 - Getting familiar with shellcode

Utilizando os arquivos fornecidos, conseguimos executar codigo binário, injetado dentro de um programa .c A partir dessa injecao abrimos shell code em 32 e 64 bits.

Transformamos código maquina em bytes e injetamos como se fosse um mini programa dentro do programa principal

utilizando 

```c
int (*func()) = (int(*)())code;
```

enganamos o sistema e dizemos para tratar a cadeia de chars como se fosse parte de uma instrução executavel.

O que acontece na verdade é que essa instrucao execuda execve("/bin//sh"), criando um novo processo que toma o lugar do processo corrente, abrindo assim um terminal shell

O que isso exemplifica é que, em casos reais, em programas vulneráveis, pode ser possível injetar código maquina, como injetamos aqui em outros endereços, sobrescrevendo o programa original e executando esse novo código injetado.

Retirando o execstack da makefile, notamos que o programa falha com segmentation fault porque o endereco onde colocamos o código maquina nao está em um segmento executável.


## Task 2 - Understanding the Vulnerable Program

No sistema vulneravel, o que acontece é que é que BUF SIZE tem apenas 100 bytes de tamano, enquanto a badfile passa 517 bytes. 

## Task 3 - Launching Attack on 32-bit Program

