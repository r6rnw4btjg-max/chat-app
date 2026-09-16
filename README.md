====================================================== --
v2 ﻧﺴﺨﺔ — )Supabase / Postgres( ﺣﻜﺎوي — ﻗﺎﻋﺪة اﻟﺒﯿﺎﻧﺎت --
Supabase → SQL Editor → Run اﻧﺴﺨﻲ ھﺬا اﻟﻤﻠﻒ ﻛﺎﻣﻞ واﻟﺼﻘﯿﮫ ﻓﻲ --
====================================================== --
اﻟﻤﻠﻔﺎت اﻟﺸﺨﺼﯿﺔ )اﺳﻢ ﻛﻞ ﻣﺴﺘﺨﺪم ﻣﺮﺑﻮط ﺑﺤﺴﺎﺑﮫ اﻟﻤﺠﮭﻮل( )1 --
create table if not exists profiles (
id uuid primary key references auth.users(id) on delete cascade,
username text not null unique,
created_at timestamptz default now()
;)
اﻟﻐﺮف )2 --
create table if not exists rooms (
id uuid primary key default gen_random_uuid(),
name text not null,
description text,
emoji text default ' ',
created_by uuid references auth.users(id),
created_at timestamptz default now()
;)
اﻟﺮﺳﺎﺋﻞ )3 --
create table if not exists messages (
id uuid primary key default gen_random_uuid(),
room_id uuid references rooms(id) on delete cascade,
user_id uuid references auth.users(id),
username text not null,
body text not null,
created_at timestamptz default now()
;)
اﻹﻋﺠﺎﺑﺎت )4 --
create table if not exists likes (
id uuid primary key default gen_random_uuid(),
message_id uuid references messages(id) on delete cascade,
user_id uuid references auth.users(id),
created_at timestamptz default now(),
unique (message_id, user_id)
;)
اﻟﺤﻈﺮ )ﻛﻞ ﻣﺴﺘﺨﺪم ﯾﺤﻈﺮ ﺛﺎﻧﻲ( )5 --
)auth.users(id وﻟﯿﺲ )profiles(id ﻣﺮﺑﻮﻃﯿﻦ ﺑـ blocker_id و blocked_id :ﻣﻼﺣﻈﺔ --
))profiles:blocked_id(username( ﺗﻠﻘﺎﺋﻲ join ﯾﻘﺪر ﯾﺴﻮي PostgREST ﻋﺸﺎن --
create table if not exists blocks (
id uuid primary key default gen_random_uuid(),
blocker_id uuid references profiles(id) on delete cascade,
blocked_id uuid references profiles(id) on delete cascade,
created_at timestamptz default now(),
unique (blocker_id, blocked_id)
;)
اﻟﺒﻼﻏﺎت )6 --
create table if not exists reports (
id uuid primary key default gen_random_uuid(),
reporter_id uuid references auth.users(id),
target_type text check (target_type in ('message','room','user')),
target_id uuid,
reason text,
created_at timestamptz default now()
;)
====================================================== --
إﻟﺰاﻣﻲ ﺣﺘﻰ ﻣﺎ أﺣﺪ ﯾﻘﺪر ﯾﻌﺪل ﺑﯿﺎﻧﺎت ﻏﯿﺮه — )RLS( ﺗﻔﻌﯿﻞ أﻣﺎن اﻟﺼﻔﻮف --
====================================================== --
alter table profiles enable row level security;
alter table rooms enable row level security;
alter table messages enable row level security;
alter table likes enable row level security;
alter table blocks enable row level security;
alter table reports enable row level security;
اﻟﻜﻞ ﯾﻘﺪر ﯾﻘﺮأ اﻟﻤﻠﻔﺎت اﻟﺸﺨﺼﯿﺔ واﻟﻐﺮف واﻟﺮﺳﺎﺋﻞ واﻹﻋﺠﺎﺑﺎت )ﻣﺤﺘﻮى ﻋﺎم( --
create policy "read profiles" on profiles for select using (true);
create policy "read rooms" on rooms for select using (true);
create policy "read messages" on messages for select using (true);
create policy "read likes" on likes for select using (true);
ﻛﻞ ﻣﺴﺘﺨﺪم ﯾﻘﺪر ﯾﺴﻮي ﺣﺴﺎﺑﮫ ﺑﺲ --
create policy "insert own profile" on profiles for insert with check (auth.uid() = id);
create policy "update own profile" on profiles for update using (auth.uid() = id);
ّﻞ )ﺣﺘﻰ ﻣﺠﮭﻮل( ﯾﻘﺪر ﯾﻨﺸﺊ ﻏﺮﻓﺔ وﯾﻜﺘﺐ رﺳﺎﻟﺔ ﺑﺎﺳﻤﮫ --
أي ﻣﺴﺘﺨﺪم ﻣﺴﺠ
create policy "create rooms" on rooms for insert with check (auth.uid() is not null);
create policy "create messages" on messages for insert with check (auth.uid() = user_id);
اﻹﻋﺠﺎب: ﻛﻞ وﺣﺪ ﯾﺘﺤﻜﻢ ﺑﺈﻋﺠﺎﺑﮫ ھﻮ ﺑﺲ --
create policy "like/unlike own" on likes for insert with check (auth.uid() = user_id);
create policy "remove own like" on likes for delete using (auth.uid() = user_id);
اﻟﺤﻈﺮ واﻟﺒﻼغ: ﻛﻞ وﺣﺪ ﯾﺸﻮف وﯾﺘﺤﻜﻢ ﺑﺤﻈﺮه وﺑﻼﻏﺎﺗﮫ ھﻮ ﺑﺲ --
create policy "manage own blocks" on blocks for all
using (auth.uid() = blocker_id) with check (auth.uid() = blocker_id);
create policy "create own reports" on reports for insert with check (auth.uid() = reporter_id
====================================================== --
ﻋﻠﻰ اﻟﺮﺳﺎﺋﻞ واﻹﻋﺠﺎﺑﺎت واﻟﻐﺮف )Realtime( ﺗﻔﻌﯿﻞ اﻻﺗﺼﺎل اﻟﻠﺤﻈﻲ --
ًﺎ ﻋﻨﺪ أي ﻣﺴﺘﺨﺪم "rooms" ﻣﻼﺣﻈﺔ: أﺿﻔﻨﺎ --
ھﻨﺎ ﻋﺸﺎن ﻗﺎﺋﻤﺔ اﻟﻐﺮف ﺗﺘﺤﺪث ﻟﺤﻈﯿ
====================================================== --
alter publication supabase_realtime add table messages;
alter publication supabase_realtime add table likes;
alter publication supabase_realtime add table rooms;