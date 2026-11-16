# Planejador Financeiro por Etapas da Vida

Produto web leve em **um único arquivo `index.html`** (HTML + CSS + JavaScript puro; Tailwind, fonte, Supabase e PostHog via CDN/snippet; sem build, sem instalação). Imagens ao lado do arquivo: `fase-40.jpg`, `fase-50.jpg`, `fase-60.jpg` (900px de largura, qualidade ~82, < 250 KB cada; ao trocar, manter esse padrão e usar caminhos relativos `./arquivo`), `og-image.png`, `favicon.png`. Obs.: o painel de pré-visualização do app mostra arquivos locais como `data:` e por isso não carrega imagens relativas; abra o `index.html` no navegador ou use o site publicado para vê-las.

## Funil guiado (wizard) — uma pergunta por tela
Barra de progresso ("Passo X de 4") nos passos 1–4, com botões "Voltar" e "Continuar →". Enter avança.

1. **Home:** tag "Simulador Gratuito • Leva 2 Minutos", título "Projete sua independência financeira etapa por etapa", subtítulo "Estamos aqui para simplificar sua independência financeira, sem planilhas complexas você vai planejar sua aposentadoria.", campos **Nome** e **Idade atual**. Deve caber em 1366x768 sem rolagem. Visitante entra direto, sem login; botão discreto "Entrar" no canto superior direito.
2. **Patrimônio atual:** Patrimônio Atual Total no Brasil (Ações, Renda Fixa e Cripto) em **R$** e fora do Brasil em **US$**. Máscara monetária; zero permitido.
3. **Poupança mensal:** "Quanto você consegue poupar por mês? (em R$)", máscara monetária.
4. **Perfil de investidor:** 2 cards — **Muito Conservador** (100% Renda Fixa) e **Moderado** (80% Renda Fixa + 20% Ações no Brasil). Obrigatório escolher.
5. **Transição:** animação de cálculo (2,5 s ou botão "Ver meu resultado") com a mensagem "Agora deixa com a gente. Você quer ter mais conforto e tranquilidade na próxima etapa da sua vida. A gente tá aqui pra te mostrar o que te espera."
6. **Resultado (próximos 15 anos):** `idadeFinal = idadeAtual + 15`; imagem: `idadeFinal <= 44` → `fase-40.jpg`; 45–54 → `fase-50.jpg`; `>= 55` → `fase-60.jpg`. Título "Nos seus {idadeFinal} anos de idade:", bloco **Patrimônio no Brasil (R$)**, bloco **Patrimônio no Exterior (USD)**, resumo: Patrimônio Total Consolidado (US$ 1 = R$ 6,30) e Nível de Dolarização (% do patrimônio em dólar). Botões: **Salvar meu teste**, **Falar com um especialista**, **Recomeçar o teste**.

## Motor de cálculo (função pura `simular`, 180 meses)
- Câmbio fixo USD/BRL = **6,30**.
- Poupança mensal: 50% fica em BRL no Brasil; 50% é convertido para USD (`(poupança × 0,5) / 6,30`) todo mês.
- Exterior (USD): **7% a.a.** (independe do perfil).
- Brasil (BRL): Muito Conservador = **10% a.a.**; Moderado = 20% × 12% + 80% × 10% = **10,4% a.a.**
- Taxa mensal: `Math.pow(1 + r_anual, 1/12) - 1`. Cada carteira iterada separadamente: `saldo = saldo * (1 + r_mensal) + aporte` (aporte ao final do mês).
- Patrimônio inicial em R$ entra na carteira Brasil; o em US$ entra na carteira exterior.
- Total consolidado em R$ = finalBRL + finalUSD × 6,30; dolarização = finalUSD × 6,30 ÷ total.
- Simulação ilustrativa, não é recomendação de investimento.

## Ações finais
- **Salvar meu teste:** logado → grava direto em `simulacoes`; deslogado → abre Criar Conta/Entrar e, após o acesso, salva automaticamente. Toast "Simulação salva com sucesso!".
- **Falar com um especialista:** abre `SPECIALIST_URL` (constante no JS; hoje `https://wa.me/5521988108887`; pode ser trocada por outro WhatsApp ou link de calendário) com `text=` pré-preenchido (nome, patrimônio em R$ e em US$).
- **Recomeçar o teste:** limpa o estado e volta ao passo 1.

## Supabase
- `@supabase/supabase-js` via CDN. Projeto `https://wrdxqdprndpqnbveeoxn.supabase.co` (Publishable Key no `index.html`).
- Login/cadastro (abas "Entrar" e "Criar Conta"; `signInWithPassword`, `signUp`, `signOut`; erros em português) só é pedido ao clicar em "Entrar" ou em "Salvar meu teste" deslogado.
- Barra do usuário (logado): nome/e-mail, "Minhas Simulações" e "Sair". "Minhas Simulações" lista do mais recente ao mais antigo (`select` filtrado por `user_id`) e reabre o resultado salvo.
- Linha gravada: `user_id`, `nome`, `contato` (e-mail da conta), `patrimonio_final_brl` (total consolidado) e `dados_simulacao` (`versao: 2` com `inputs`, `premissas`, `resultado`). Registros antigos (por etapas, sem `versao`) aparecem como "formato antigo" e não reabrem.
- Pré-requisito: rodar `supabase-setup.sql` (coluna `user_id`, `created_at`, policies RLS).

## Tracking (PostHog)
Snippet no `<head>` (não alterar). Eventos: `step_1_completed`, `step_2_completed`, `step_3_completed`, `step_4_completed`, `simulation_viewed`, `cta_save_clicked`, `cta_specialist_clicked`, `cta_restart_clicked`. Propriedades: `perfil_investidor`, `idade_final`, `percentual_dolar`. **Nunca** enviar nome, e-mail ou valores de patrimônio/poupança.

## Publicação (Git/GitHub)
- Repositório remoto: https://github.com/BROscale/planejadorfinanceiro.git (remote `origin`, branch `main`).
- **Sempre que o usuário pedir para atualizar ou publicar o site, fazer commit e push direto para a branch `main`** (sem criar branch nem PR).
- Git instalado em `C:\Program Files\Git\cmd\git.exe` (pode não estar no PATH de sessões já abertas).
