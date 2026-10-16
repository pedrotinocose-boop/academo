# Academo

Clube do livro sem spoiler. Flutter (Android + web) + Supabase.

## O que já funciona (fase 1)

- Conta: cadastro, login, Google, recuperar senha, onboarding (nome, @ com checagem de disponibilidade, gêneros, ritmo)
- Explorar clubes com busca e filtros; página do clube; entrar (aberto, com aprovação ou só convite, com limite de vagas)
- Criar clube em 3 passos: o cronograma por semana é gerado no banco
- Hoje: meta do dia, progresso, pedidos pendentes
- Marcar onde parei (+/−, controle deslizante, atalhos)
- Fórum por clube, estilo Reddit: tópicos com etiqueta, votos, ordenação, respostas aninhadas, apagar, denunciar
- **Proteção contra spoiler calculada no servidor**: o conteúdo além da sua página não sai do banco até você chegar lá (ou pedir para revelar)
- Pessoas: aprovar pedidos, tornar moderadora, remover (permissões checadas no banco)
- Perfis, seguir, bloquear, notificações (respostas e novos seguidores), configurações, sair

O banco já tem as tabelas de mensagens privadas, denúncias, repasses e transações para as próximas fases.

## Rodar pela primeira vez (Windows)

1. **Flutter**: instale em https://docs.flutter.dev/get-started/install/windows (stable) e rode `flutter doctor`.
2. **Gere as pastas de plataforma** (só uma vez), dentro desta pasta:
   ```
   flutter create . --platforms=android,web --project-name academo --org app.academo
   flutter pub get
   ```
3. **Supabase**: crie um projeto em https://supabase.com. Depois, no **SQL Editor**, rode na ordem:
   - `supabase/migrations/20261013000000_schema.sql`
   - `supabase/seed.sql`

   (Ou, com a CLI: `supabase link --project-ref SEU_REF` e `supabase db push`.)
4. **Chaves**: copie `env.example.json` para `env.json` e preencha com *Project Settings › API* (URL e anon key).
5. **Rodar**:
   ```
   flutter run -d chrome --dart-define-from-file=env.json
   flutter run -d <seu-android> --dart-define-from-file=env.json
   ```

### Ajustes no painel do Supabase
- *Authentication › Providers*: ligue Google se quiser o botão (Client ID do Google Cloud).
- *Authentication › URL Configuration*: adicione `http://localhost:*` e o domínio do app web nas URLs de redirecionamento.
- Para testar rápido, desligue *Confirm email* (o app também trata o fluxo com confirmação).

## Estrutura

```
lib/
  app/        tema, rotas e sessão
  data/       modelos, acesso ao Supabase (repo.dart) e providers (Riverpod)
  widgets/    capa, avatar, estados vazios e de erro, diálogos
  features/   auth, onboarding, today, explore, club, forum, profile, shell
supabase/
  migrations/ tabelas, gatilhos, funções (RPC) e regras de acesso (RLS)
  seed.sql    livros iniciais
```

## Fase 4 — livros, leitura pessoal e organização

- **Busca de livros** (`/books/search`): acervo do Academo + Google Livros + Open Library. `GOOGLE_BOOKS_KEY` no env.json é opcional (sem ela funciona, com limite menor).
- **Ficha do livro** (`/book/:id`): sinopse, nota média, avaliações, clubes que leram e "Quero ler / Lendo / Lido".
- **Leitura pessoal**: marcar páginas fora de clube; conta na sequência de dias e aparece no Hoje.
- **Explorar**: escolha da semana, amigos lendo, combina com você, começam esta semana, livros em alta.
- **Meus clubes** (aba aparece para quem cria/modera): pendências, métricas e **Ganhos** (saldo, saque via Pix, plano de apoio, extrato).

### Ativar pagamentos (Pix via Pagar.me)

```
npm i -g supabase
supabase login
supabase link --project-ref SEU-PROJETO
supabase secrets set PAGARME_SECRET_KEY=sk_... ACADEMO_FEE_PERCENT=15
supabase functions deploy create-checkout
supabase functions deploy pagarme-webhook --no-verify-jwt
```
No painel da Pagar.me, aponte o webhook para `https://SEU-PROJETO.supabase.co/functions/v1/pagarme-webhook`.

## Fase 5 e 6

- Chat privado em ordem cronológica (mais recente embaixo).
- Estante com qualquer livro já lido, separando "em clube" e "por conta própria".
- **Meus clubes** (aba própria para quem cria/modera) e **painel por clube** (`/club/:id/manage`): Painel, Pessoas, Moderação, Agenda, Ganhos e Ajustes.
- Marcar leitura **por página ou por capítulo** (a página continua sendo a medida interna; capítulos são convertidos proporcionalmente).

## Fase 7

- Modo leitura com cronômetro (`/read`), minutos na aba Hoje.
- Votação do próximo livro e clube contínuo (novas edições com as mesmas pessoas).
- Encontros: link da chamada, "Pôr na agenda" (Google Agenda) e lista de presença.
- Aviso para o clube inteiro; apoio mensal; página pública do clube com compartilhar (`APP_URL` no env.json).
- Silenciar pessoas, filtro de palavrões e limites anti-spam (no banco).
- Perfil público com clubes e estante; busca de pessoas e leitores parecidos (`/people`).
- Excluir clube e tirar qualquer livro da estante.

## Ainda falta

- Push e lembretes de leitura (precisa configurar Firebase no Android).
- Leitor de código de barras para ISBN (precisa permissão de câmera).
- Stripe para apoios internacionais.

## Catálogo de livros (busca)

A busca mostra primeiro o **catálogo do Academo** (tabela `books`, função `search_books`): sem acento, tolera erro de digitação e acha por título, autoria, **editora** ou ISBN, ordenando por parecença + popularidade. Google Livros / Open Library entram só embaixo, como complemento (sem artigos/revistas e sem edições em francês, espanhol etc.).

Montar/atualizar o catálogo inicial (~1.100 títulos em `tool/catalogo/titulos.txt`):

1. `dart run tool/catalogo/build_catalog.dart` — busca a edição brasileira de cada título (capa, editora, ISBN) usando a `GOOGLE_BOOKS_KEY` do env.json e gera `supabase/migrations/20261014050000_catalogo_seed.sql`. Para em 900 consultas para não estourar a cota diária (~1.000); se parar, rode de novo no dia seguinte que ele continua de onde parou (`tool/catalogo/cache.json`).
2. No SQL Editor do Supabase, rode `20261014040000_catalogo.sql` e depois `20261014050000_catalogo_seed.sql`.

Para incluir livros: acrescente linhas `Título | Autor` em `titulos.txt` e repita os passos (o seed não duplica).

## Ajustes de 15/10

Rode no SQL Editor: `supabase/migrations/20261015000000_ajustes.sql`.

- Estante: "Não lembro quando li" funciona (a regra do banco foi refeita) e marcar **Lido** na ficha do livro pergunta o ano.
- Perfil: tocar em **seguidores** ou **seguindo** abre a lista (`/person/:id/follows`).
- Clubes com **etiquetas** (literatura brasileira, clássicos, não ficção, ensaios, HQs, encontros presenciais…): escolhidas ao criar/editar o clube, aparecem nos cards, viram filtros no Explorar e entram na busca. Lista em `lib/data/club_tags.dart`.
- Meus clubes: as dicas do fim da página têm um X para fechar.
- Busca na internet: só Google Livros e só edições em português (título e capa brasileiros); estrangeiras só se não houver nenhuma em português. Capas que faltam também vêm só da edição brasileira do Google. Se a cota da chave do Google acabar, o app tenta sem a chave. A versão anterior ficou em `tool/book_api_antes_2026-10-15.dart.txt`.
- Catálogo: +300 clássicos em `titulos.txt` (logo depois da prioridade). Dica: crie uma segunda chave do Google Livros e coloque em `GOOGLE_BOOKS_KEY_CATALOGO` no env.json — assim o script não gasta a cota da busca do app.
