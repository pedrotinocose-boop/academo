# Academo na Play Store — passo a passo

Tudo que já está pronto no projeto:

- `android/app/build.gradle.kts` assina o release com a chave de upload (`android/key.properties`) e mira a API 36 (exigência da Play para apps novos).
- Nome do app no celular: **Academo**. Ícone novo (normal + adaptativo) em `android/app/src/main/res`.
- `play_store/icone_512.png` (ícone da loja) e `play_store/grafico_destaque_1024x500.png` (gráfico de destaque).
- `site/`: Política de privacidade, Termos de uso e página de exclusão de conta — **precisam estar no ar** (passo 2).
- Links para Termos e Privacidade no cadastro e em Configurações.
- No Android, o botão **Apoiar (Pix)** fica escondido (`paymentsInApp` em `lib/app/links.dart`). Pagamento por fora do Google Play Faturamento por acesso/benefício digital é motivo de reprovação. Na web continua igual.
- O app já tem: excluir conta, denunciar comentário, bloquear e silenciar pessoas — exigências para apps com conteúdo de usuários.

---

## 1. Criar a chave de upload (uma vez só — GUARDE ESTE ARQUIVO E AS SENHAS)

```
cd C:\Users\Pedro\Desktop\Projects\academo
keytool -genkey -v -keystore C:\Users\Pedro\academo-upload.jks -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```
(`keytool` vem com o Android Studio: `C:\Program Files\Android\Android Studio\jbr\bin\keytool.exe` se o comando não for achado.)

Crie `android\key.properties` (já está no .gitignore):
```
storePassword=SENHA_QUE_VOCE_ESCOLHEU
keyPassword=SENHA_QUE_VOCE_ESCOLHEU
keyAlias=upload
storeFile=C:\\Users\\Pedro\\academo-upload.jks
```
Faça backup do `.jks` (Drive, pendrive). Com a Assinatura de apps do Google Play ativa, se perder dá para pedir troca, mas dá trabalho.

## 2. Pôr as páginas legais no ar

A Play exige URL pública da Política de privacidade e da exclusão de conta.
- Se `academo.app` já é seu: suba o conteúdo de `site/` na raiz (fica `https://academo.app/privacidade.html`).
- Se não: Netlify Drop (https://app.netlify.com/drop) — arraste a pasta `site` e pronto; ou GitHub Pages. Depois coloque no env.json:
  `"LEGAL_URL": "https://SEU-ENDERECO"` para os links do app apontarem para lá.
- **Antes:** troque `[SEU NOME COMPLETO]` em `site/privacidade.html` e `site/termos.html`.
- Confirme que o e-mail **contato@academo.app** recebe mensagens (está no app e nas páginas).

## 3. Gerar o pacote (.aab)

Suba a versão em `pubspec.yaml` a cada envio (`version: 1.0.0+1`, depois `1.0.1+2`… o número depois do + sempre aumenta).
```
cd C:\Users\Pedro\Desktop\Projects\academo
flutter clean
flutter pub get
flutter build appbundle --release --dart-define-from-file=env.json
```
Arquivo: `build\app\outputs\bundle\release\app-release.aab`.
Se reclamar de SDK 36: Android Studio › SDK Manager › instale "Android 16 (API 36)".
Teste antes num celular: `flutter run --release --dart-define-from-file=env.json` (login com e-mail, login com Google, entrar em clube, fórum, chat, excluir conta de teste).

## 4. Supabase para produção

- *Authentication › URL Configuration*: inclua `app.academo://login-callback` nas Redirect URLs (volta do login com Google no Android).
- Ative *Confirm email* e personalize os e-mails (remetente, texto em português).
- Denúncias: veja a tabela `reports` no painel de vez em quando — a Play exige que **você** também modere, não só os organizadores.

## 5. Play Console

1. Conta de desenvolvedor: https://play.google.com/console (taxa única de US$ 25; verificação de identidade com documento). Conta **pessoal** criada depois de nov/2023 precisa do teste fechado do passo 7 antes de produção.
2. **Criar app**: nome "Academo", idioma pt-BR, App, Gratuito.
3. Painel › *Configurar seu app* — preencha tudo:
   - **Política de privacidade**: URL da `privacidade.html`.
   - **Acesso ao app**: "Todas ou algumas funcionalidades são restritas" → crie uma conta de teste (e-mail + senha, já com onboarding feito e dentro de um clube) e informe aqui. Sem isso o revisor não entra e reprova.
   - **Anúncios**: Não.
   - **Classificação de conteúdo** (questionário IARC): categoria "Social ou comunicação"; usuários interagem/trocam mensagens: Sim; compartilha localização: Não (cidade é texto livre opcional); compras digitais: Não; sem violência, sexo, drogas, jogos de azar.
   - **Público-alvo**: 16–17 e 18+ (bate com os Termos). Não marque menores de 13.
   - **Segurança dos dados**: respostas abaixo.
   - **Exclusão de conta**: URL da `excluir-conta.html`.
   - **Apps do governo / financeiros / saúde**: Não. **Notícias**: Não.
4. **Página da loja principal**: textos abaixo, `icone_512.png`, `grafico_destaque_1024x500.png` e de 2 a 8 capturas de tela de celular (1080×1920 ou 1080×2400, PNG/JPG). Tire do app rodando: Hoje, Explorar, página do clube, fórum sem spoiler, perfil/estante.
5. Categoria: **Livros e referências**. E-mail de contato: contato@academo.app.

## 6. Segurança dos dados — respostas

- Coleta ou compartilha dados? **Sim**. Criptografados em trânsito? **Sim**. Usuário pode pedir exclusão? **Sim**.
- Coletados (todos **não compartilhados** para fins de terceiros; processados pelo Supabase como prestador de serviço não conta como compartilhamento):

| Tipo | Item | Obrigatório? | Finalidade |
|---|---|---|---|
| Informações pessoais | Nome | Sim | Funcionalidade do app, gerenciamento da conta |
| Informações pessoais | Endereço de e-mail | Sim | Gerenciamento da conta, comunicações |
| Informações pessoais | IDs do usuário (@) | Sim | Funcionalidade, gerenciamento da conta |
| Informações pessoais | Endereço (cidade, texto livre) | Opcional | Funcionalidade (personalização) |
| Fotos e vídeos | Fotos (foto de perfil) | Opcional | Funcionalidade |
| Mensagens | Outras mensagens no app (chat e fórum) | Opcional | Funcionalidade |
| Atividade no app | Interações no app; Outro conteúdo gerado pelo usuário | Sim | Funcionalidade, personalização |
| Atividade no app | Histórico de pesquisa no app (busca de livros) | Opcional | Funcionalidade |
| Informações e desempenho do app | Registros de falhas / diagnóstico | Sim | Análise, segurança (logs do servidor) |

- Informações financeiras: **não** na versão Android (o Pix está escondido). Se um dia ativar pagamentos no Android, atualize.
- Coleta é efêmera? Não. Pode ser desativada? só onde marquei "Opcional".

## 7. Teste fechado (obrigatório para conta pessoal nova)

1. *Testar › Teste fechado* › criar faixa › enviar o `.aab` › notas da versão.
2. Testadores: lista de e-mails (contas Google) — **mínimo 12 pessoas inscritas, por 14 dias seguidos**. Chame amigos, grupos de leitura, a lista de espera do Academo. Coloque uns 15–20 para ter folga (se alguém sair, a contagem pode quebrar).
3. Envie a cada testador o link de inscrição (aparece na faixa) + o link da Play Store; eles precisam **aceitar o teste e instalar**, e de preferência abrir o app alguns dias.
4. Durante os 14 dias, mande pelo menos uma atualização (versão +1) corrigindo algo que os testadores apontarem — ajuda na aprovação.
5. Depois dos 14 dias: *Painel › Solicitar acesso à produção* — responde perguntas sobre o teste (quantas pessoas, que feedback teve, o que mudou). Responda com detalhes.
6. Aprovado → *Produção* › criar versão com o mesmo .aab (ou mais novo) › lançamento. Revisão leva de horas a ~7 dias.

(Teste interno, até 100 pessoas, sem revisão: bom para você e 2–3 amigos testarem já, mas não conta para os 14 dias.)

## 8. Textos da loja

**Nome (30):** Academo: clube do livro

**Descrição curta (80):** Leia junto, no seu ritmo, com fórum por semana e zero spoiler.

**Descrição completa:**
```
Ler sozinha é bom. Ler junto é melhor — desde que ninguém estrague o final.

O Academo é o clube do livro sem spoiler. Cada clube tem um cronograma por semana e um fórum que só mostra o que você já leu. Chegou na página 120? Você vê as conversas até a página 120. O resto fica trancado até você chegar lá.

O QUE DÁ PARA FAZER
• Entrar em clubes abertos ou por convite — ou criar o seu em 3 passos
• Marcar onde parou (por página ou capítulo) e ver sua meta do dia
• Conversar no fórum do clube, por semana, com votos e respostas
• Modo leitura com cronômetro e sequência de dias lendo
• Estante pessoal: quero ler, lendo, lido, com avaliações
• Votar no próximo livro do clube e marcar encontros (presenciais ou online)
• Seguir leitores parecidos com você e trocar mensagens
• Compartilhar seu progresso nos stories

FEITO PARA COMUNIDADES
Organiza um clube? Tenha painel com pessoas, pedidos para entrar, moderação de denúncias, agenda e avisos para todo mundo.

SEGURO
Denuncie comentários, bloqueie ou silencie qualquer pessoa e escolha quem pode te mandar mensagem.

Comece hoje: escolha um livro, entre num clube e leia junto.
```

**Notas da versão 1.0:** Primeira versão do Academo: clubes do livro sem spoiler, fórum por semana, estante e modo leitura.

## Motivos comuns de reprovação (já cobertos — não desfaça)
- Sem conta de teste em "Acesso ao app".
- Política de privacidade fora do ar, genérica ou sem citar os dados reais.
- Sem exclusão de conta no app **e** por URL.
- Conteúdo de usuário sem denúncia/bloqueio ou sem termos que proíbam abuso.
- Pagamento digital por fora do Google Play.
- Ícone/nome parecido com marca de terceiros, capturas de tela que não são do app.
- Formulário de Segurança dos dados diferente do que o app faz.
