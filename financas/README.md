# Plano Financeiro

App de finanças pessoais em um único arquivo (`index.html`), no estilo iOS, para acompanhar um plano de 12 meses no dia a dia.

## O que faz

- **Assistente (topo da tela Hoje)**: escreva ou fale pelo ditado do teclado (“gastei 32 no almoço no Nubank e 18 de Uber ontem”) e o app lança cada item, com data, categoria e cartão. Também entende contas fixas pagas, faturas, recebimentos, compras parceladas e dinheiro guardado na caixinha. Depois recalcula tudo e avisa quanto ainda dá para gastar hoje e até o fim do mês, com opção de desfazer.
- **Extrato do mês**: aceita PDF (inclusive com senha), CSV, OFX, print da tela ou texto colado. Classifica cada movimentação (gasto, conta fixa, recorrente, fatura, recebimento, caixinha, estorno, tarifa), marca o que parece repetido, lança o que você aprovar e gera uma análise: categorias, onde mais gastou, tarifas, assinaturas, delivery, Pix no crédito, parcelas e uma leitura de especialista feita pela IA. Extrato com “compra no débito”, Pix e boletos é tratado como conta, mesmo que o banco também tenha cartão (a conta Nubank não vira cartão Nubank).
- **Posso comprar?** e **Perguntar**: simulam o impacto de uma compra no mês e respondem perguntas sobre o plano.

- **Hoje**: quanto ainda dá para gastar por dia no mês, próximos vencimentos e recebimentos (com marcação de pago/recebido), alertas de saldo negativo e o gráfico do gasto livre acumulado.
- **Diário**: lançamento rápido do gasto livre, com categoria e forma de pagamento. Para compras no cartão, mostra em qual fatura a compra cai.
- **Mês**: recebimentos, contas fixas, faturas, extras e contas anuais do mês, com valores reais e o fechamento (sobra para o Pote Anuais e para a Reserva).
- **Plano**: caixinhas ao longo do plano, sobra e entradas × saídas por mês, tabela linha a linha, faturas previstas × teto, calendário do saldo dia a dia e cenários de gasto diário.
- **Mais**: 11 calculadoras (patrimônio em 2 anos, juros compostos, meta, parcelamento de um extra, à vista × parcelado, em qual fatura cai, rendimento da caixinha, reserva de emergência, custo de um hábito, custo do limite, IPVA/licenciamento), além de premissas editáveis, tutorial, glossário, pendências e backup.

## IA e leitor automático

Dentro do claude.ai, o app usa a IA do artifact (capacidade `sample`) para entender a fala, ler extratos e prints e escrever análises. Sem a IA, ou se ela falhar, entra um leitor automático em português que funciona offline: números falados, datas (“ontem”, “sexta”, “dia 5”), cartões, categorias por palavras-chave, parser de CSV, OFX e linhas de extrato, e o pdf.js (carregado do cdnjs, com o jsDelivr como segunda fonte). A página não tem acesso ao microfone; para falar, usa-se o ditado do próprio teclado.

## Site próprio com nuvem (Lovable)

O mesmo arquivo também roda como site normal. Um carregador pequeno (projeto Lovable) serve `app.html` e define `window.PF_CONFIG` com a URL e a chave pública do banco (Supabase/Lovable Cloud). Com isso, o app entra no **modo nuvem**:

- Na primeira vez, a pessoa cria uma **chave pessoal** (20 caracteres) ou digita uma que já tem.
- Cada documento é criptografado no navegador com AES-GCM de 256 bits. A chave AES é derivada da chave pessoal com PBKDF2-SHA256 (150 mil iterações), e o dono é o SHA-256 da chave. O banco guarda só texto cifrado.
- O banco não aceita acesso direto (RLS sem políticas). O app usa apenas as funções `pf_pull`, `pf_put` e `pf_purge` (security definer), que exigem o hash de dono.
- Há uma cópia local para funcionar offline e uma fila de envio que sobe quando a internet volta. A sincronização roda a cada 20 segundos, ao voltar para a aba e ao reconectar.
- Em Mais → Backup e nuvem: ver a chave, trocar de chave (recriptografa tudo), sair do aparelho e sincronizar agora.

## Como os dados são guardados

- Publicado como artifact do claude.ai, o app usa o banco de dados do próprio artifact. As regras deixam leitura e escrita só para o dono, e os dados sincronizam entre celular e computador.
- Aberto fora do claude.ai, cai no modo local, com os dados guardados no `localStorage` do navegador. Os backups podem ser exportados e importados em JSON.

O arquivo não contém nenhum dado pessoal: os valores do plano são carregados no banco depois da publicação ou restaurados de um backup.

## Modelo de cálculo

- Sobra do mês = entradas − (contas fixas + cobranças recorrentes no cartão + faturas antigas + gasto livre + extras) − colchão (só no 1º mês).
- A compra no cartão conta no mês em que é feita (competência). As faturas aparecem à parte, pela data de vencimento.
- Fatura antes do plano: em Cartões, “Já devido no 1º mês” guarda o valor da fatura atual e a data em que foi conferido. A fatura mostra quais compras do Diário e cobranças recorrentes já estão dentro desse valor e quanto falta lançar. Compras no cartão lançadas depois da conferência e antes do plano entram por cima, na fatura e nas saídas do 1º mês.
- No mês atual, o resto do mês é previsto pelo que o app manda gastar por dia, sem passar da regra: quem gasta mais no começo (uma viagem, por exemplo) e segura depois não aparece estourando a sobra.
- Nas Premissas, “Gasto livre previsto sai” escolhe entre débito (sai da conta no dia) e um cartão (entra na fatura e sai da conta no vencimento). Isso muda o calendário e as faturas previstas.
- A sobra enche primeiro o Pote Anuais, só com o necessário para as contas anuais até o fim do plano, e o resto vai para a Reserva. As duas rendem a taxa mensal das premissas.
- Sobra negativa sai primeiro das caixinhas. O que elas não cobrem sai do saldo da conta (o colchão diminui), e o calendário não inventa um resgate que não existe.
- Os meses já passados usam o gasto real do Diário. Os saldos reais informados no fechamento substituem a previsão.
