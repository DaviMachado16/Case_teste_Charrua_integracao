# Case_teste_Charrua_integracao
Case de Teste e Análise Técnica: Integração de Sistema de Retaguarda vs. Portal de Fidelidade (API)
Este repositório contém uma análise técnica detalhada realizada durante a validação de uma integração entre um sistema de ERP (Retaguarda) e um Portal de Fidelidade Externo. O objetivo deste documento é demonstrar metodologias de teste de caixa-preta e caixa-cinza, isolamento de bugs via API e documentação de fluxos lógicos.

🛠️ Tecnologias e Ferramentas Utilizadas
Postman: Validação de endpoints REST e simulação de payloads.

Log Analysis: Investigação de traces de erro e monitoramento de requisições.

Jira: Documentação dos desenvolvedores para entedimento de fluxogramas e lógica de decisão.

Metodologia de Teste: Identificação de inconsistências entre documentação técnica (QP) e comportamento em produção.

📋 Cenário 1: Indisponibilidade Crítica da "Carga Full"
Descrição do Problema
O botão de sincronização total (Carga Full) — responsável por parear tabelas de prazos e produtos — encontra-se desabilitado na interface de retaguarda, impedindo a atualização de dados essenciais no portal externo.

Investigação de Causa Raiz
Ao analisar a documentação de implementação (QP-34597), identificou-se uma regra de negócio que desabilita o botão após o primeiro processamento bem-sucedido. Contudo, em casos de falha parcial na primeira tentativa, o sistema entra em um "estado de bloqueio", onde o botão permanece inativo e os dados no portal ficam inconsistentes.

Validação via API (Isolamento do Erro)
Para confirmar se o problema era de comunicação (Backend) ou de interface (Frontend), realizei um POST manual via Postman:

Endpoint: POST /cadastro/forma-pagamento

Payload de Teste:

JSON

{
  "acao": "I", 
  "empresa": "XXXXXXXXXXXXXX", 
  "codigo": 12147,
  "valor": 5.5,
  "descricao": "Dinheiro"
}
Resultado: O portal processou a requisição com sucesso (HTTP 200), confirmando que os Endpoints e as Credenciais estão operacionais, isolando a falha na lógica de estado do botão na Retaguarda.

📋 Cenário 2: Loop Infinito e Falha no Fluxo de Checkout (PDV)
Descrição do Problema
Na unidade filial, o acionamento da rotina de carga causa um travamento na interface (loop infinito). Paralelamente, no Ponto de Venda (PDV), o sistema não processa o retorno de descontos de fidelidade, impedindo o fechamento da venda.

Fluxograma de Análise Logística
Snippet de código

graph TD
    A[Início da Venda PDV] --> B[Aplicação de Voucher]
    B --> C{Chamada API Fidelidade}
    C -- Sucesso --> D[Retorno de Desconto]
    D --> E{Processamento Local PDV}
    E -- Erro --> F[Interface Congelada / Loop]
    C -- Falha --> G[Erro de Timeout]
Metodologia de Depuração
Extração de Body: Capturei a montagem do JSON de "Inicializar Venda" gerado pelo PDV.

Simulação Externa: Executei o mesmo processo via Postman com um voucher ativo.

Conclusão: A API externa devolveu o objeto de desconto corretamente. O erro foi identificado na camada de consumo do JSON pelo PDV, que não consegue interpretar o retorno quando há múltiplas regras de preço aplicadas.
