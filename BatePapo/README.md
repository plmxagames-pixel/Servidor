# BatePapo — MXDrive

Página de bate-papo criada na pasta `BatePapo/` do repositório Servidor.

## Ativar as mensagens em tempo real

A página usa o projeto Supabase já configurado no MXDrive para autenticação. Para que todos vejam as mesmas mensagens, é preciso criar a tabela e ativar o Realtime uma vez:

1. Abra o painel do projeto Supabase: https://supabase.com/dashboard/project/ehvgixekasdmzuceyjye
2. Entre em **SQL Editor** e execute este SQL:

```sql
create table if not exists public.chat_messages (
  id bigint generated always as identity primary key,
  user_id uuid not null references auth.users(id) on delete cascade,
  display_name text not null default 'Usuário',
  content text not null check (char_length(content) between 1 and 1500),
  created_at timestamptz not null default now()
);

alter table public.chat_messages enable row level security;

create policy "Usuários autenticados podem ler mensagens"
on public.chat_messages for select to authenticated
using (true);

create policy "Usuários autenticados podem enviar suas próprias mensagens"
on public.chat_messages for insert to authenticated
with check (auth.uid() = user_id);

grant select, insert on public.chat_messages to authenticated;
grant usage, select on sequence public.chat_messages_id_seq to authenticated;

alter publication supabase_realtime add table public.chat_messages;
```

3. Se o último comando disser que a tabela já está na publicação, ignore apenas esse aviso.
4. Em **Authentication → URL Configuration**, confira se a URL do site está autorizada. Para GitHub Pages, adicione a URL pública da página do chat à lista de **Redirect URLs**, se necessário.
5. Publique o repositório e abra `https://plmxagames-pixel.github.io/Servidor/BatePapo/`.

## Observações

- O acesso exige uma conta Supabase/MXDrive. Usuários novos podem precisar confirmar o e-mail antes de entrar.
- O chat é uma sala pública para usuários autenticados; não envie senhas ou dados privados.
- A chave publishable usada no frontend é própria para clientes públicos; as permissões reais são limitadas pelas políticas RLS acima. Nunca coloque uma chave `service_role` no site.
