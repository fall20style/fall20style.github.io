---
title: discord.js의 Client 이벤트 목록
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - discord
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

## 1. 필수 및 가장 자주 사용되는 이벤트
봇의 기본 동작을 위해 거의 무조건 사용되는 이벤트임.

* InteractionCreate: 유저가 슬래시 명령어, 버튼, 메뉴 등을 사용할 때 발생함. (가장 중요함)
* ClientReady: 봇이 로그인을 완료하고 준비가 끝났을 때 발생함. (once로 주로 사용함)
* MessageCreate: 채널에 새로운 메시지가 올라왔을 때 발생함. (텍스트 명령어 대응 시 사용함)

``` typescript
import { Client, Events, GatewayIntentBits } from 'discord.js';

// 1. 클라이언트 생성 (필요한 인텐트 설정)
const client = new Client({
    intents: [
        GatewayIntentBits.Guilds,           // 서버 관련 기본 동작
        GatewayIntentBits.GuildMessages,    // 메시지 수신용
        GatewayIntentBits.MessageContent,   // 메시지 내용 읽기용 (설정 필요)
    ],
});

// 2. ClientReady: 봇 기동 시 딱 한 번 실행됨
client.once(Events.ClientReady, (readyClient) => {
    console.log(`✅ ${readyClient.user.tag} 로 로그인 성공함!`);
});

// 3. InteractionCreate: 슬래시 명령어 등 처리함
client.on(Events.InteractionCreate, async (interaction) => {
    if (!interaction.isChatInputCommand()) return;

    if (interaction.commandName === '출석') {
        await interaction.reply('출석 체크 완료함!');
    }
});

// 4. MessageCreate: 일반 채팅 메시지 감지함
client.on(Events.MessageCreate, (message) => {
    // 봇이 쓴 메시지는 무시함
    if (message.author.bot) return;

    if (message.content === '안녕') {
        message.reply('반가움!');
    }
});

// 봇 로그인함
client.login(process.env.DISCORD_TOKEN);
```

## 2. 유저 및 멤버 관리 관련
서버 관리나 환영 봇을 만들 때 필수적인 이벤트임.

* GuildMemberAdd: 새로운 유저가 서버에 입장했을 때 발생함.
* GuildMemberRemove: 유저가 서버를 나갔을 때 발생함.
* GuildMemberUpdate: 유저의 닉네임 변경, 역할 추가/제거 시 발생함.
* PresenceUpdate: 유저의 상태(온라인, 게임 중 등)가 변할 때 발생함.

``` typescript
import { Client, Events, GatewayIntentBits } from 'discord.js';

const client = new Client({
    intents: [
        GatewayIntentBits.Guilds,
        GatewayIntentBits.GuildMembers,  // 멤버 입장/퇴장/수정 감지용 (필수)
        GatewayIntentBits.GuildPresences // 온라인 상태/게임 감지용 (필수)
    ],
});

// 1. GuildMemberAdd: 새로운 유저가 서버에 들어왔을 때 실행함
client.on(Events.GuildMemberAdd, (member) => {
    console.log(`👋 새 멤버 입장함: ${member.user.tag}`);
    // 특정 채널에 환영 메시지를 보낼 수 있음
});

// 2. GuildMemberRemove: 유저가 서버를 나갔을 때 실행함
client.on(Events.GuildMemberRemove, (member) => {
    console.log(`😭 멤버 나감: ${member.user.tag}`);
});

// 3. GuildMemberUpdate: 닉네임이나 역할이 바뀌었을 때 실행함
client.on(Events.GuildMemberUpdate, (oldMember, newMember) => {
    // 닉네임이 변경되었는지 확인함
    if (oldMember.nickname !== newMember.nickname) {
        console.log(`📝 닉네임 변경됨: ${oldMember.nickname} -> ${newMember.nickname}`);
    }
    
    // 역할(Role)이 추가되거나 제거되었는지 확인함
    if (oldMember.roles.cache.size !== newMember.roles.cache.size) {
        console.log(`🎖️ 역할 상태가 업데이트됨: ${newMember.user.tag}`);
    }
});

// 4. PresenceUpdate: 온라인/오프라인 상태나 활동이 변할 때 실행함
client.on(Events.PresenceUpdate, (oldPresence, newPresence) => {
    const user = newPresence?.user;
    if (!user) return;

    console.log(`🎮 ${user.tag}의 상태가 [${oldPresence?.status}]에서 [${newPresence.status}]로 변함`);
});

client.login(process.env.DISCORD_TOKEN);
```

## 3. 서버(길드) 및 채널 관리
서버 설정이나 채널 구성이 바뀔 때 사용함.

* ChannelCreate / ChannelDelete: 채널이 생성되거나 삭제되었을 때 발생함.
* GuildCreate / GuildDelete: 봇이 서버에 초대받거나 쫓겨났을 때 발생함.
* RoleCreate / RoleDelete / RoleUpdate: 역할이 생성, 삭제, 수정되었을 때 발생함.

``` typescript
import { Client, Events, GatewayIntentBits } from 'discord.js';

const client = new Client({
    intents: [
        GatewayIntentBits.Guilds, // 채널, 역할, 길드 이벤트 감지를 위해 필수임
    ],
});

// 1. GuildCreate / GuildDelete: 봇의 서버 출입 감지함
client.on(Events.GuildCreate, (guild) => {
    console.log(`🏰 새로운 서버에 초대받음: ${guild.name} (ID: ${guild.id})`);
});

client.on(Events.GuildDelete, (guild) => {
    console.log(`🚫 서버에서 나감 혹은 서버가 삭제됨: ${guild.name}`);
});

// 2. ChannelCreate / ChannelDelete: 채널 변화 감지함
client.on(Events.ChannelCreate, (channel) => {
    console.log(`📁 새 채널 생성됨: ${channel.name}`);
});

client.on(Events.ChannelDelete, (channel) => {
    console.log(`🗑️ 채널 삭제됨: ${channel.name}`);
});

// 3. RoleCreate / RoleDelete / RoleUpdate: 역할 관련 이벤트 처리함
client.on(Events.RoleCreate, (role) => {
    console.log(`🆕 새 역할 만들어짐: ${role.name}`);
});

client.on(Events.RoleDelete, (role) => {
    console.log(`🔥 역할 사라짐: ${role.name}`);
});

client.on(Events.RoleUpdate, (oldRole, newRole) => {
    // 역할 이름이 바뀌었는지 확인함
    if (oldRole.name !== newRole.name) {
        console.log(`🏷️ 역할 이름 변경됨: ${oldRole.name} -> ${newRole.name}`);
    }
    // 색상이 바뀌었는지 확인함
    if (oldRole.color !== newRole.color) {
        console.log(`🎨 ${newRole.name} 역할의 색상이 변경됨`);
    }
});

client.login(process.env.DISCORD_TOKEN);

```

## 4. 메시지 상세 동작
메시지 삭제나 수정 등 세밀한 관리가 필요할 때 사용함.

* MessageUpdate: 유저가 메시지를 수정했을 때 발생함.
* MessageDelete: 메시지가 삭제되었을 때 발생함.
* MessageReactionAdd / MessageReactionRemove: 메시지에 이모지 반응을 달거나 뗐을 때 발생함.

``` typescript
import { Client, Events, GatewayIntentBits, Partials } from 'discord.js';

const client = new Client({
    intents: [
        GatewayIntentBits.Guilds,
        GatewayIntentBits.GuildMessages,
        GatewayIntentBits.MessageContent,
        GatewayIntentBits.GuildMessageReactions, // 반응 감지용 필수임
    ],
    // 메시지가 봇 구동 전에 작성되었어도 이벤트를 받으려면 Partial 설정이 필요함
    partials: [Partials.Message, Partials.Channel, Partials.Reaction],
});

// 1. MessageUpdate: 메시지가 수정되었을 때 실행함
client.on(Events.MessageUpdate, (oldMessage, newMessage) => {
    if (oldMessage.partial) return; // 이전 내용이 캐시에 없으면 무시함
    if (oldMessage.content === newMessage.content) return; // 내용 변화 없으면 종료함

    console.log(`📝 메시지 수정됨: "${oldMessage.content}" -> "${newMessage.content}"`);
});

// 2. MessageDelete: 메시지가 삭제되었을 때 실행함
client.on(Events.MessageDelete, (message) => {
    console.log(`🗑️ 메시지 삭제됨: ${message.content}`);
});

// 3. MessageReactionAdd: 이모지 반응을 달았을 때 실행함
client.on(Events.MessageReactionAdd, async (reaction, user) => {
    // 봇이 단 반응은 무시함
    if (user.bot) return;

    // 만약 반응이 partial 상태라면 완전한 데이터를 가져옴 (fetch)
    if (reaction.partial) {
        try {
            await reaction.fetch();
        } catch (error) {
            console.error('반응 데이터를 가져오는데 실패함:', error);
            return;
        }
    }

    console.log(`${user.tag}님이 ${reaction.emoji.name} 반응을 달았음`);
});

// 4. MessageReactionRemove: 이모지 반응을 취소했을 때 실행함
client.on(Events.MessageReactionRemove, (reaction, user) => {
    console.log(`${user.tag}님이 ${reaction.emoji.name} 반응을 제거함`);
});

client.login(process.env.DISCORD_TOKEN);

```

## 5. 음성 및 시스템 관련
음악 봇이나 시스템 모니터링 시 필요함.

* VoiceStateUpdate: 음성 채널 입장/퇴장, 마이크 상태 변화 시 발생함.
* Error: 봇 실행 중 에러가 발생했을 때 발생함. (크래시 방지용으로 사용함)
* Debug: 봇 내부의 상세 동작 로그를 확인할 때 사용함.

## Related Posts
- [디스코드 출석체크 봇 2 - Event Driven Architecture]({% link _posts/2026-04-26-discord-bot-event-driven-arch.md %})
- [discord-bot-gemini 4.22 수정]({% link _posts/2026-04-22-discord-bot-gemini-fix.md %})
- [discord.js의 Interaction 타입]({% link _posts/2026-04-25-discord-js-interacraction-button.md %})
- [Docker sandbox 실행 + Discord Bot 통합]({% link _posts/2026-04-12-docker-sandboxing-discord-bot.md %})
- [Discord Echo 봇 환경 구축 및 실행]({% link _posts/2026-04-12-typescript-discord-echo-bot.md %})
- [Discord 매 10분 자동 메시지 전송 기능 구현]({% link _posts/2026-04-19-discord-alerts-every-10mins.md %})
- [Discord bot 환경 변수 적용 및 타입 에러를 수정]({% link _posts/2026-04-19-discord-bot-env-in-code.md %})
- [디스코드 봇 개발 주제 모음]({% link _posts/2026-04-19-discord-bot-toy-project-list.md %})
- [디스코드 출석체크 봇 1 - 초기설정, 로그인 기능]({% link _posts/2026-04-24-my-attendance-bot-step1.md %})
- [디스코드 출석체크 봇 4 - db기능 통합하고 모듈화]({% link _posts/2026-04-26-prisma-db-handling.md %})
- [디스코드 출석체크 봇 3 - db기능만 떼어서 실험]({% link _posts/2026-04-26-prisma-db-init-test.md %})
