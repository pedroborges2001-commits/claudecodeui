# Plano Financeiro

App de finanças pessoais em um único arquivo (`index.html`), no estilo iOS, para acompanhar um plano de 12 meses no dia a dia.

## O que faz

- **Hoje**: quanto ainda dá para gastar por dia no mês, próximos vencimentos e recebimentos (com marcação de pago/recebido), alertas de saldo negativo e o gráfico do gasto livre acumulado.
- **Diário**: lançamento rápido do gasto livre, com categoria e forma de pagamento. Para compras no cartão, mostra em qual fatura a compra cai.
- **Mês**: recebimentos, contas fixas, faturas, extras e contas anuais do mês, com valores reais e o fechamento (sobra para o Pote Anuais e para a Reserva).
- **Plano**: caixinhas ao longo do plano, sobra e entradas × saídas por mês, tabela linha a linha, faturas previstas × teto, calendário do saldo dia a dia e cenários de gasto diário.
- **Mais**: 11 calculadoras (patrimônio em 2 anos, juros compostos, meta, parcelamento de um extra, à vista × parcelado, em qual fatura cai, rendimento da caixinha, reserva de emergência, custo de um hábito, custo do limite, IPVA/licenciamento), além de premissas editáveis, tutorial, glossário, pendências e backup.

## Como os dados são guardados

- Publicado como artifact do claude.ai, o app usa o banco de dados do próprio artifact. As regras deixam leitura e escrita só para o dono, e os dados sincronizam entre celular e computador.
- Aberto fora do claude.ai, cai no modo local, com os dados guardados no `localStorage` do navegador. Os backups podem ser exportados e importados em JSON.

O arquivo não contém nenhum dado pessoal: os valores do plano são carregados no banco depois da publicação ou restaurados de um backup.

## Modelo de cálculo

- Sobra do mês = entradas − (contas fixas + cobranças recorrentes no cartão + faturas antigas + gasto livre + extras) − colchão (só no 1º mês).
- A compra no cartão conta no mês em que é feita (competência). As faturas aparecem à parte, pela data de vencimento.
- A sobra enche primeiro o Pote Anuais, só com o necessário para as contas anuais até o fim do plano, e o resto vai para a Reserva. As duas rendem a taxa mensal das premissas.
- Os meses já passados usam o gasto real do Diário. Os saldos reais informados no fechamento substituem a previsão.
