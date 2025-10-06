# 🚀 Desafio DIO: Orquestração Serverless com AWS Step Functions

Este repositório documenta minha experiência e aprendizado na criação e consolidação de workflows automatizados, utilizando o **AWS Step Functions**. O projeto visa cumprir o desafio da Digital Innovation One (DIO), demonstrando a capacidade de orquestrar serviços *serverless* da AWS de forma robusta e resiliente.

| **Status do Projeto** | **Concluído e Documentado** |
| :--- | :--- |
| **Tecnologias Focadas** | AWS Step Functions, AWS Lambda, Amazon States Language (ASL) |

---

## 1. Fundamentos: Entendendo o AWS Step Functions

O AWS Step Functions é um serviço de orquestração *serverless* que permite construir aplicativos distribuídos usando **máquinas de estado (State Machines)**. Ele ajuda a gerenciar a lógica complexa, o tratamento de erros e a coordenação de fluxos de trabalho que envolvem múltiplos serviços AWS.

### Componentes Chave
* **Máquina de Estado (State Machine):** O coração do workflow, onde a sequência de passos é definida.
* **Estados (States):** Os blocos de construção do workflow. Os principais usados nesta prática foram:
    * `Task`: Executa uma unidade de trabalho (ex: chama uma função Lambda).
    * `Choice`: Adiciona lógica de ramificação (IF/ELSE) baseada na saída do estado anterior.
    * `Succeed`/`Fail`: Estados terminais que indicam o sucesso ou falha da execução.
* **ASL (Amazon States Language):** Linguagem baseada em JSON/YAML usada para definir a estrutura da State Machine.

---

## 2. O Workflow Automatizado Construído

O objetivo deste laboratório foi simular um fluxo de trabalho comum em arquiteturas *serverless*, onde o resultado de uma função determina o caminho a ser seguido.

### Objetivo do Workflow

O workflow simula o **processamento e validação de uma transação financeira**. Ele executa uma função Lambda que determina um status (`APROVADA` ou `REPROVADA`) e usa um estado `Choice` para tomar a decisão final, garantindo que o fluxo termine com sucesso ou falha de forma clara.

### Fluxo de Execução (Passo a Passo)

1.  **Início:** A Máquina de Estado é acionada com o *payload* da transação (ex: ID e valor).
2.  **`Task State` (Chamada Lambda):** O estado **`ValidarTransacao`** invoca uma função AWS Lambda. Esta função **simula a checagem de risco/fraude e retorna o status da transação em um campo `$.status`**.
3.  **`Choice State` (Decisão):** O estado **`DecidirStatus`** avalia o campo `$.status` retornado pela Lambda.
4.  **Ramificação:**
    * Se o status for **APROVADO** (`StringEquals: APROVADA`), o fluxo segue para o estado **`TransacaoAprovada`**.
    * Caso contrário (se for `REPROVADA`), o fluxo segue o caminho `Default` para o estado **`TransacaoReprovada`**.
5.  **Finalização:** O workflow termina em um dos estados terminais (`Succeed` ou `Fail`).

### Diagrama Visual do Workflow

Aqui está uma representação visual do workflow que construí no console da AWS:

![Diagrama do Step Functions mostrando o fluxo de Task, Choice e Succeed/Fail](./images/workflow-diagrama.png)

---

## 3. Detalhes Técnicos e Código

### Definição da State Machine (ASL)

O núcleo do workflow é o código JSON/ASL. O trecho abaixo foca na lógica de decisão implementada no estado `Choice`, que analisa a saída da Lambda:

```json
{
  "Comment": "Workflow de Validação de Transação usando Step Functions e Lambda.",
  "StartAt": "ValidarTransacao",
  "States": {
    
    "ValidarTransacao": {
      "Type": "Task",
      "Resource": "[ARN_DA_SUA_LAMBDA]",
      "Next": "DecidirStatus"
    },
    
    "DecidirStatus": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.status",
          "StringEquals": "APROVADA",
          "Next": "TransacaoAprovada"
        }
      ],
      "Default": "TransacaoReprovada"
    },
    
    "TransacaoAprovada": {
      "Type": "Succeed"
    },
    
    "TransacaoReprovada": {
      "Type": "Fail"
    }
  }
}
