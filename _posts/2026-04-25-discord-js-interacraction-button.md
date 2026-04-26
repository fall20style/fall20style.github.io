---
title: discord.js의 Interaction 타입
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

- Interaction은 유저가 봇과 상호작용하는 모든 행위를 담는 가장 큰 단위의 타입

## 1. ChatInputCommandInteraction (슬래시 명령어)

유저가 /를 입력해서 명령어를 실행했을 때의 타입임. 가장 많이 쓰임.

* 핵심 속성: commandName (명령어 이름), options (유저가 입력한 인자값)
* 확인법: `interaction.isChatInputCommand()`

## 2. ButtonInteraction (버튼)

메시지에 달린 버튼을 유저가 클릭했을 때 발생함.

* 핵심 속성: customId (버튼을 구분하기 위해 개발자가 설정한 ID)
* 확인법: `interaction.isButton()`

``` typescript
import { 
    Client, 
    Events, 
    GatewayIntentBits, 
    ActionRowBuilder, 
    ButtonBuilder, 
    ButtonStyle, 
    Interaction 
} from 'discord.js';

const client = new Client({ intents: [GatewayIntentBits.Guilds] });

client.on(Events.InteractionCreate, async (interaction: Interaction) => {
    
    // 1. 슬래시 명령어 '/버튼'을 입력받았을 때
    if (interaction.isChatInputCommand()) {
        if (interaction.commandName === '버튼') {
            // 버튼 생성함
            const button = new ButtonBuilder()
                .setCustomId('confirm_button') // 버튼을 식별할 고유 ID (중요!)
                .setLabel('여기를 누르셈!')     // 버튼에 적힐 글자
                .setStyle(ButtonStyle.Primary); // 버튼 색상 (파란색)

            // 버튼을 가로 줄(ActionRow)에 담음
            const row = new ActionRowBuilder<ButtonBuilder>()
                .addComponents(button);

            // 메시지와 함께 버튼 전송함
            await interaction.reply({
                content: '아래 버튼을 눌러보셈!',
                components: [row]
            });
        }
    }

    // 2. 버튼 클릭을 감지했을 때 (ButtonInteraction)
    if (interaction.isButton()) {
        // 위에서 설정한 customId로 어떤 버튼인지 구별함
        if (interaction.customId === 'confirm_button') {
            // 버튼을 누른 유저에게 답장함
            await interaction.reply({ 
                content: `✅ **${interaction.user.username}**님이 버튼을 클릭함!`, 
                ephemeral: true // 누른 사람에게만 보이게 설정함 (선택사항)
            });
        }
    }
});

client.login(process.env.DISCORD_TOKEN);
```


## 3. AnySelectMenuInteraction (선택 메뉴)

드롭다운 메뉴에서 항목을 선택했을 때 발생함.

* 핵심 속성: values (유저가 선택한 항목들의 배열), customId
* 확인법: `interaction.isAnySelectMenu()`

## 4. ModalSubmitInteraction (모달 창)

봇이 띄워준 입력창(모달)에 유저가 내용을 적고 '제출'을 눌렀을 때 발생함.

* 핵심 속성: fields (유저가 입력한 텍스트 값들)
* 확인법: `interaction.isModalSubmit()`

## 5. AutocompleteInteraction (자동 완성)

유저가 명령어를 입력하는 도중, 봇이 추천 검색어를 보여줄 때 발생함.

* 주의점: reply를 쓸 수 없고 respond를 써야 함.
* 확인법: `interaction.isAutocomplete()`

## 모든 Interaction의 공통 속성

타입이 무엇이든 상관없이 공통적으로 쓸 수 있는 것들임.

* user: 상호작용을 시도한 유저 객체임.
* guild: 상호작용이 일어난 서버(길드) 정보임.
* channel: 상호작용이 일어난 채널 정보임.
* reply(): 유저에게 답장을 보내는 핵심 메서드임.


## 한눈에 보는 요약표

| 타입 | 실제 행위 | 확인 메서드 |
|---|---|---|
| ChatInput | /명령어 입력 | .isChatInputCommand() |
| Button | 버튼 클릭 | .isButton() |
| SelectMenu | 드롭다운 선택 | .isAnySelectMenu() |
| Modal | 입력 폼 제출 | .isModalSubmit() |
| UserContext | 유저 우클릭 앱 실행 | .isUserContextMenuCommand() |

## Related Posts
- [디스코드 출석체크 봇 2 - Event Driven Architecture]({% link _posts/2026-04-26-discord-bot-event-driven-arch.md %})
- [discord-bot-gemini 4.22 수정]({% link _posts/2026-04-22-discord-bot-gemini-fix.md %})
- [discord.js의 Client 이벤트 목록]({% link _posts/2026-04-25-discord-js-event-list.md %})
- [Docker sandbox 실행 + Discord Bot 통합]({% link _posts/2026-04-12-docker-sandboxing-discord-bot.md %})
- [Discord Echo 봇 환경 구축 및 실행]({% link _posts/2026-04-12-typescript-discord-echo-bot.md %})
- [Discord 매 10분 자동 메시지 전송 기능 구현]({% link _posts/2026-04-19-discord-alerts-every-10mins.md %})
- [Discord bot 환경 변수 적용 및 타입 에러를 수정]({% link _posts/2026-04-19-discord-bot-env-in-code.md %})
- [디스코드 봇 개발 주제 모음]({% link _posts/2026-04-19-discord-bot-toy-project-list.md %})
- [디스코드 출석체크 봇 1 - 초기설정, 로그인 기능]({% link _posts/2026-04-24-my-attendance-bot-step1.md %})
- [디스코드 출석체크 봇 4 - db기능 통합하고 모듈화]({% link _posts/2026-04-26-prisma-db-handling.md %})
- [디스코드 출석체크 봇 3 - db기능만 떼어서 실험]({% link _posts/2026-04-26-prisma-db-init-test.md %})
