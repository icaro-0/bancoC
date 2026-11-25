# bancoC


#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define MAX 100
#define HIST 100
#define CORRENTE 1
#define SALARIO 2
#define POUPANCA 3

typedef struct {
    int numero, tipo;
    float saldo;
} Conta;

typedef struct {
    char descricao[80];
    float valor;
} Movimento;

typedef struct {
    char nome[80];
    Conta contas[3];
    Movimento historico[HIST];
    int qtdContas, qtdHist;
    int emprestimo, titulo;
} Cliente;

typedef struct {
    Cliente clientes[MAX];
    int qtd, proximo;
    int aberto, sorteado, vencedor;
    float premio;
} Banco;

// FUNCOES AUXILIARES

int conta(Cliente *c, int tipo) {
    for (int i = 0; i < c->qtdContas; i++)
        if (c->contas[i].tipo == tipo) return i;
    return -1;
}

float saldo(Cliente *c) {
    float total = 0;
    for (int i = 0; i < c->qtdContas; i++)
        total += c->contas[i].saldo;
    return total;
}

void registrar(Cliente *c, char *desc, float valor) {
    if (c->qtdHist < HIST) {
        strcpy(c->historico[c->qtdHist].descricao, desc);
        c->historico[c->qtdHist++].valor = valor;
    }
}

void mostrarContas(Cliente *c) {
    for (int i = 0; i < c->qtdContas; i++)
        printf("%04d-%02d | Saldo: R$ %.2f\n",
            c->contas[i].numero,
            c->contas[i].tipo,
            c->contas[i].saldo);
}

// CADASTRO

void abrirBanco(Banco *b) {
    if (b->aberto) {
        printf("Banco ja esta aberto!\n");
        return;
    }

    printf("Quantidade de clientes: ");
    scanf("%d", &b->qtd);
    getchar();

    if (b->qtd < 1 || b->qtd > MAX) {
        printf("Quantidade invalida!\n");
        b->qtd = 0;
        return;
    }

    for (int i = 0; i < b->qtd; i++) {
        Cliente *c = &b->clientes[i];

        printf("\nNome do cliente: ");
        fgets(c->nome, 80, stdin);
        c->nome[strcspn(c->nome, "\n")] = '\0';

        c->qtdContas = c->qtdHist = 0;
        c->emprestimo = c->titulo = 0;

        int n;
        printf("Quantidade de contas (1 a 3): ");
        scanf("%d", &n);

        if (n < 1 || n > 3) {
            printf("Quantidade invalida. Cliente sem contas adicionais.\n");
            n = 1;
        }

        for (int j = 0; j < n; j++) {
            int tipo;

            printf("\n01-Corrente | 02-Salario | 03-Poupanca\n");
            printf("Tipo da conta: ");
            scanf("%d", &tipo);

            if (tipo < 1 || tipo > 3 ||
                conta(c, tipo) != -1 ||
                (tipo == CORRENTE && conta(c, SALARIO) != -1) ||
                (tipo == SALARIO && conta(c, CORRENTE) != -1)) {
                printf("Conta invalida ou combinacao proibida!\n");
                j--;
                continue;
            }

            Conta *ct = &c->contas[c->qtdContas++];
            ct->numero = b->proximo++;
            ct->tipo = tipo;
            ct->saldo = 0;

            printf("Conta criada: %04d-%02d\n",
                ct->numero, tipo);
        }
        getchar();
    }

    b->aberto = 1;
    printf("\nBanco aberto com sucesso!\n");
}

// MOVIMENTACOES

void movimentar(Cliente *c, int saque) {
    mostrarContas(c);

    int tipo;
    printf("Tipo da conta: ");
    scanf("%d", &tipo);

    int i = conta(c, tipo);
    if (i == -1) {
        printf("Conta inexistente!\n");
        return;
    }

    float valor;
    printf("Valor: R$ ");
    scanf("%f", &valor);

    if (valor <= 0) {
        printf("Valor invalido!\n");
        return;
    }

    if (saque && c->contas[i].saldo < valor) {
        printf("Saldo insuficiente!\n");
        return;
    }

    c->contas[i].saldo += saque ? -valor : valor;
    registrar(c, saque ? "Saque" : "Deposito", valor);

    printf("Operacao realizada! Saldo: R$ %.2f\n",
        c->contas[i].saldo);
}

// EMPRESTIMO

void emprestimo(Cliente *c) {
    int co = conta(c, CORRENTE);
    int po = conta(c, POUPANCA);

    if (co == -1 || po == -1 || c->contas[po].saldo <= 0 ||
        c->emprestimo) {
        printf("Cliente nao esta apto ao emprestimo!\n");
        return;
    }

    float limite = saldo(c) * 2.5, valor;

    printf("Limite disponivel: R$ %.2f\n", limite);
    printf("Valor solicitado: R$ ");
    scanf("%f", &valor);

    if (valor <= 0 || valor > limite) {
        printf("Valor fora do limite!\n");
        return;
    }

    c->contas[co].saldo += valor;
    c->emprestimo = 1;
    registrar(c, "Emprestimo", valor);

    printf("Emprestimo aprovado!\n");
}

// TITULO DE CAPITALIZACAO

void titulo(Cliente *c) {
    int i = conta(c, SALARIO);

    if (i == -1 || c->titulo ||
        c->contas[i].saldo < 1000 ||
        c->contas[i].saldo > 1500) {
        printf("Cliente nao esta apto a adquirir titulo!\n");
        return;
    }

    c->titulo = 1;
    registrar(c, "Titulo de capitalizacao adquirido", 0);
    printf("Titulo adquirido com sucesso!\n");
}

// MENU DO CLIENTE

void menuCliente(Banco *b) {
    if (!b->aberto) {
        printf("Abra o banco primeiro!\n");
        return;
    }

    int id;
    printf("Numero do cliente (1 a %d): ", b->qtd);
    scanf("%d", &id);

    if (id < 1 || id > b->qtd) {
        printf("Cliente inexistente!\n");
        return;
    }

    Cliente *c = &b->clientes[id - 1];

    printf("\nCliente: %s | Saldo total: R$ %.2f\n",
        c->nome, saldo(c));

    int co = conta(c, CORRENTE);
    if (co != -1 && c->contas[co].saldo < 0)
        printf("ALERTA: SUA CONTA CORRENTE ESTA NEGATIVA!\n");

    int op;
    do {
        printf("\n1-Contas  2-Deposito  3-Saque\n");
        printf("4-Emprestimo  5-Titulo  0-Voltar\n");
        printf("Opcao: ");
        scanf("%d", &op);

        switch (op) {
            case 1: mostrarContas(c); break;
            case 2: movimentar(c, 0); break;
            case 3: movimentar(c, 1); break;
            case 4: emprestimo(c); break;
            case 5: titulo(c); break;
            case 0: break;
            default: printf("Opcao invalida!\n");
        }
    } while (op != 0);
}

// HISTORICO DO BANCO

void historico(Banco *b) {
    if (!b->aberto) {
        printf("Banco fechado!\n");
        return;
    }

    float total = 0;

    for (int i = 0; i < b->qtd; i++)
        total += saldo(&b->clientes[i]);

    printf("\n== SALDO TOTAL DO BANCO ==\n");
    printf("%d | R$ %.2f\n", b->qtd, total);

    printf("\n== CLIENTES APTOS A EMPRESTIMOS ==\n");
    for (int i = 0; i < b->qtd; i++) {
        Cliente *c = &b->clientes[i];
        int co = conta(c, CORRENTE), po = conta(c, POUPANCA);

        if (co != -1 && po != -1 &&
            c->contas[po].saldo > 0 && !c->emprestimo)
            printf("%s | %04d-01 | R$ %.2f\n",
                c->nome, c->contas[co].numero, saldo(c) * 2.5);
    }

    printf("\n== CLIENTES COM SALDO NEGATIVO NA CONTA CORRENTE ==\n");
    for (int i = 0; i < b->qtd; i++) {
        Cliente *c = &b->clientes[i];
        int co = conta(c, CORRENTE);

        if (co != -1 && c->contas[co].saldo < 0)
            printf("%s | %04d-01 | R$ %.2f\n",
                c->nome, c->contas[co].numero,
                c->contas[co].saldo);
    }

    printf("\n== CLIENTES APTOS A COMPRA DE TITULOS ==\n");
    for (int i = 0; i < b->qtd; i++) {
        Cliente *c = &b->clientes[i];
        int sa = conta(c, SALARIO);

        if (sa != -1 && !c->titulo &&
            c->contas[sa].saldo >= 1000 &&
            c->contas[sa].saldo <= 1500)
            printf("%s | %04d-02 | R$ %.2f\n",
                c->nome, c->contas[sa].numero,
                c->contas[sa].saldo);
    }

    printf("\n== HISTORICO DE MOVIMENTACOES ==\n");
    for (int i = 0; i < b->qtd; i++) {
        Cliente *c = &b->clientes[i];
        printf("\n%s:\n", c->nome);

        for (int j = 0; j < c->qtdHist; j++)
            printf("%s | R$ %.2f\n",
                c->historico[j].descricao,
                c->historico[j].valor);
    }

    printf("\n== CLIENTE CONTEMPLADO NO SORTEIO ==\n");
    if (b->sorteado)
        printf("%s | Premio: R$ %.2f\n",
            b->clientes[b->vencedor].nome, b->premio);
    else
        printf("Nenhum sorteio realizado.\n");
}

// SORTEIO FIBONACCI

int fibonacci() {
    int a = 0, b = 1, n = time(NULL) % 20 + 1;

    for (int i = 0; i < n; i++) {
        int temp = a + b;
        a = b;
        b = temp;
    }

    return a % 5;
}

void sorteio(Banco *b) {
    if (!b->aberto || b->qtd < 5) {
        printf("Banco fechado ou menos de cinco clientes!\n");
        return;
    }

    int ordem[MAX];

    for (int i = 0; i < b->qtd; i++)
        ordem[i] = i;

    // Ordenacao por saldo total
    for (int i = 0; i < b->qtd - 1; i++)
        for (int j = i + 1; j < b->qtd; j++)
            if (saldo(&b->clientes[ordem[j]]) >
                saldo(&b->clientes[ordem[i]])) {
                int temp = ordem[i];
                ordem[i] = ordem[j];
                ordem[j] = temp;
            }

    printf("\n== PARTICIPANTES ==\n");

    for (int i = 0; i < 5; i++) {
        Cliente *c = &b->clientes[ordem[i]];

        printf("%s | R$ %.2f\n", c->nome, saldo(c));

        if (!c->titulo) {
            printf("Sorteio cancelado: participante sem titulo!\n");
            return;
        }
    }

    int pos = fibonacci();
    int id = ordem[pos];
    Cliente *c = &b->clientes[id];

    // Premio baseado no saldo total anterior ao sorteio
    float premio = saldo(c) * 10;

    // Credita o premio na conta de maior saldo
    int maior = 0;
    for (int i = 1; i < c->qtdContas; i++)
        if (c->contas[i].saldo > c->contas[maior].saldo)
            maior = i;

    c->contas[maior].saldo += premio;
    registrar(c, "Premio do sorteio", premio);

    b->sorteado = 1;
    b->vencedor = id;
    b->premio = premio;

    printf("\n== VENCEDOR DO SORTEIO ==\n");
    printf("%s | Premio: R$ %.2f\n", c->nome, premio);
}

// PROGRAMA PRINCIPAL

int main() {
    Banco b = {0};
    b.proximo = 1000;

    int op;

    do {
        printf("\n===== BANCO DIGITAL =====\n");
        printf("1 - Abrir banco\n");
        printf("2 - Consultar historico\n");
        printf("3 - Efetuar sorteio\n");
        printf("4 - Acessar cliente\n");
        printf("0 - Sair\n");
        printf("Opcao: ");
        scanf("%d", &op);

        switch (op) {
            case 1: abrirBanco(&b); break;
            case 2: historico(&b); break;
            case 3: sorteio(&b); break;
            case 4: menuCliente(&b); break;
            case 0: printf("Sistema encerrado!\n"); break;
            default: printf("Opcao invalida!\n");
        }
    } while (op != 0);

    return 0;
}
