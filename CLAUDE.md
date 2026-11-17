# Planejador Financeiro por Etapas da Vida

Produto web leve em **um único arquivo `index.html`** (HTML + CSS + JavaScript puro; Tailwind, fonte, Supabase e PostHog via CDN/snippet; sem build, sem instalação). Imagens ao lado do arquivo: `fase-40.jpg`, `fase-50.jpg`, `fase-60.jpg` (900px de largura, qualidade ~82, < 250 KB cada; ao trocar, manter esse padrão e usar caminhos relativos `./arquivo`), `og-image.png`, `favicon.png`. Obs.: o painel de pré-visualização do app mostra arquivos locais como `data:` e por isso não carrega imagens relativas; abra o `index.html` no navegador ou use o site publicado para vê-las.

## Funil guiado (wizard) — uma pergunta por tela
Barra de progresso ("Passo X de 5") nos passos 1–5, com botões "Voltar" e "Continuar →". Enter avança. Ordem das telas (ids no HTML): `step1` Home → `step2` Patrimônio → `step3` Poupança → `step4` Perfil → `step5` Renda passiva desejada → `step6` Transição → `step7` Resultado (+ `telaAuth`). A tela de resultado tem "← Voltar e editar respostas" (volta ao `step5`).

1. **Home:** tag "Simulador Gratuito • Leva 2 Minutos", título "Projete sua independência financeira etapa por etapa", subtítulo "Estamos aqui para simplificar sua independência financeira, sem planilhas complexas você vai planejar sua aposentadoria.", campos **Nome** e **Idade atual**. Deve caber em 1366x768 sem rolagem. Visitante entra direto, sem login; botão discreto "Entrar" no canto superior direito.
2. **Patrimônio atual:** Patrimônio Atual Total no Brasil (Ações, Renda Fixa e Cripto) em **R$** e fora do Brasil em **US$**. Máscara monetária; zero permitido.
3. **Poupança mensal:** "Quanto você consegue poupar por mês? (em R$)", máscara monetária.
4. **Perfil de investidor:** 2 cards — **Muito Conservador** (100% Renda Fixa) e **Moderado** (80% Renda Fixa + 20% Ações no Brasil). Obrigatório escolher.
5. **Renda passiva desejada (última pergunta):** "Qual renda passiva mensal você gostaria de ter?" / "Considere valores de hoje — vamos calcular quanto você precisa poupar por mês para alcançar esse padrão de vida sem corroer seu patrimônio." Input com máscara R$ (padrão R$ 30.000) e pills `R$ 15 mil`, `R$ 30 mil`, `R$ 60 mil`, `R$ 100 mil`. Valor deve ser > 0.
6. **Transição:** animação de cálculo (2,5 s ou botão "Ver meu resultado") com a mensagem "Agora deixa com a gente. Você quer ter mais conforto e tranquilidade na próxima etapa da sua vida. A gente tá aqui pra te mostrar o que te espera."
7. **Resultado (próximos 15 anos), 3 blocos empilhados em `max-w-[960px]`:**
   - **Bloco 1 — Resultado nominal:** `idadeFinal = idadeAtual + 15`; imagem: `idadeFinal <= 44` → `fase-40.jpg`; 45–54 → `fase-50.jpg`; `>= 55` → `fase-60.jpg`. Título "Nos seus {idadeFinal} anos de idade:", **Patrimônio no Brasil (R$)**, **Patrimônio no Exterior (USD)**, Patrimônio Total Consolidado (US$ 1 = R$ 6,30), Nível de Dolarização. Sem botões neste bloco.
   - **Bloco 2 — Insights · poder de compra real** (badge `INSIGHTS • PODER DE COMPRA REAL`, título "Quanto o seu plano atual gera de renda passiva real"): os dois cards vêm logo abaixo do título. `VP_BRL = VF_BRL/(1,045)^anos`, `VP_USD = VF_USD/(1,025)^anos`; renda = `VP × 4% / 12`. Card claro (Brasil R$) e card escuro `#0f172a` com número verde `#10b981` (Exterior US$), cada um citando o patrimônio equivalente em valor de hoje. Um indicador **`+`** fica entre os dois cards (coluna central no desktop, linha entre eles no mobile) e, abaixo, uma linha "Renda Passiva Real Total (Brasil + Exterior): R$ [renda BRL + renda USD × 6,30]/mês". A explicação metodológica (inflação reinvestida, saque só com juros reais, 4% real a.a.) fica num rodapé de 11px no fim do bloco.
   - **Bloco 3 — Planejamento reverso** (badge `PLANEJAMENTO REVERSO`): título "SUA META: R$ X mil por mês de renda passiva" (`R$ 60 mil` se múltiplo exato de mil, senão valor completo). **Caixa única** de destaque: "Poupança mensal atual: R$ Y" → "Para atingir a sua meta de renda mensal passiva, esse é o valor que você deveria poupar todo mês →" **R$ X** (grande, encerra a caixa; sem linha de divisão Brasil/exterior) (se `Y >= X`, nota positiva). Todas as premissas (patrimônio total necessário, divisão da renda entre Brasil e Exterior, perfil, horizonte, 50/50, inflações, 4% real, câmbio 6,30) ficam num único texto de 11px no fim do bloco.
   - **Final da página:** botões **Salvar meu teste**, **Falar com um especialista**, **Recomeçar o teste** e o disclaimer "Simulação ilustrativa, não é recomendação de investimento…", abaixo do Bloco 3.

## Motor de cálculo (função pura `simular`, 180 meses)
- Câmbio fixo USD/BRL = **6,30**.
- Poupança mensal: 50% fica em BRL no Brasil; 50% é convertido para USD (`(poupança × 0,5) / 6,30`) todo mês.
- Exterior (USD): **7% a.a.** (independe do perfil).
- Brasil (BRL): Muito Conservador = **10% a.a.**; Moderado = 20% × 12% + 80% × 10% = **10,4% a.a.**
- Taxa mensal: `Math.pow(1 + r_anual, 1/12) - 1`. Cada carteira iterada separadamente: `saldo = saldo * (1 + r_mensal) + aporte` (aporte ao final do mês).
- Patrimônio inicial em R$ entra na carteira Brasil; o em US$ entra na carteira exterior.
- Total consolidado em R$ = finalBRL + finalUSD × 6,30; dolarização = finalUSD × 6,30 ÷ total.
- Simulação ilustrativa, não é recomendação de investimento.

## Planejamento reverso (função pura `planejamentoReverso`) e renda real (`rendaPassivaReal`)
- Premissas: câmbio 6,30; inflação Brasil 4,5% a.a.; inflação EUA 2,5% a.a.; retirada sustentável 4% real a.a.; alocação 50% Brasil / 50% Dólar; mesmas taxas nominais do perfil escolhido (Brasil 10% ou 10,4%; exterior 7%).
- Fisher: `r_real = (1 + r_nominal)/(1 + inflação) - 1`; mensal: `(1 + r_real)^(1/12) - 1` (Brasil e EUA).
- Alvo hoje = `renda × 12 / 0,04`. Patrimônio inicial (R$ + US$ × 6,30) é projetado em termos reais por 180 meses e abatido do alvo (`Math.max(0, …)`).
- Aporte `X` (R$/mês de hoje, exibido com `Math.ceil`): `falta / (0,5·A_BR + 0,5·A_US)`, com `A = ((1+i)^n − 1)/i`; metade em R$ e metade em US$ (÷ 6,30).

## Ações finais
- **Salvar meu teste:** logado → grava direto em `simulacoes`; deslogado → abre Criar Conta/Entrar e, após o acesso, salva automaticamente. Toast "Simulação salva com sucesso!".
- **Falar com um especialista:** abre `SPECIALIST_URL` (constante no JS; hoje `https://wa.me/5521988108887`; pode ser trocada por outro WhatsApp ou link de calendário) com `text=` pré-preenchido (nome, patrimônio em R$ e em US$).
- **Recomeçar o teste:** limpa o estado e volta ao passo 1.

## Supabase
- `@supabase/supabase-js` via CDN. Projeto `https://wrdxqdprndpqnbveeoxn.supabase.co` (Publishable Key no `index.html`).
- Login/cadastro (abas "Entrar" e "Criar Conta"; `signInWithPassword`, `signUp`, `signOut`; erros em português) só é pedido ao clicar em "Entrar" ou em "Salvar meu teste" deslogado.
- Barra do usuário (logado): nome/e-mail, "Minhas Simulações" e "Sair". "Minhas Simulações" lista do mais recente ao mais antigo (`select` filtrado por `user_id`) e reabre o resultado salvo.
- Linha gravada: `user_id`, `nome`, `contato` (e-mail da conta), `patrimonio_final_brl` (total consolidado) e `dados_simulacao` (`versao: 2` com `inputs` — inclui `renda_passiva_desejada` —, `premissas`, `resultado` — inclui renda real e `poupanca_mensal_necessaria`). Registros antigos (por etapas, sem `versao`) aparecem como "formato antigo" e não reabrem.
- Pré-requisito: rodar `supabase-setup.sql` (coluna `user_id`, `created_at`, policies RLS).

## Tracking (PostHog)
Snippet no `<head>` (não alterar). Eventos: `step_1_completed` … `step_5_completed`, `simulation_viewed`, `cta_save_clicked`, `cta_specialist_clicked`, `cta_restart_clicked`. Propriedades: `perfil_investidor`, `idade_final`, `percentual_dolar`, `renda_passiva_desejada` e `poupanca_mensal_necessaria` (esta só na tela de resultado). **Nunca** enviar nome, e-mail nem os valores de patrimônio/poupança informados pelo usuário.

## Publicação (Git/GitHub)
- Repositório remoto: https://github.com/BROscale/planejadorfinanceiro.git (remote `origin`, branch `main`).
- **Sempre que o usuário pedir para atualizar ou publicar o site, fazer commit e push direto para a branch `main`** (sem criar branch nem PR).
- Git instalado em `C:\Program Files\Git\cmd\git.exe` (pode não estar no PATH de sessões já abertas).
