---
title: Python과 Exaone API로 문맥을 기억하는 Discord AI 챗봇
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - AI
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---



- Discord 봇을 만들 때 주기적으로 서버에 요청을 보내는 폴링(Polling) 방식 대신, **`discord.py`의 이벤트 기반(WebSocket) 방식**을 사용
- 메시지가 입력될 때만 반응하는 효율적인 봇을 구현

- **Exaone API**를 연동
- 채널별 이전 대화 내역을 기억하며 연속적으로 대화(Loop)할 수 있는 Discord AI 챗봇(`bot.py`) 구현 방법을 정리

---

## 1. 주요 특징
* **이벤트 기반(WebSocket) 아키텍처**: 불필요한 폴링 없이 채팅 이벤트(`on_message`)가 발생할 때만 동작합니다.
* **비동기 API 호출 (`aiohttp`)**: `requests` 대신 비동기 HTTP 클라이언트를 사용해, AI가 답변을 생성하는 동안에도 봇이 멈추지 않고 다른 명령어를 처리할 수 있습니다.
* **대화 문맥 유지 (Multi-turn Loop)**: 채널 ID별로 대화 히스토리(`messages`)를 메모리에 저장하여, 이전 질문과 답변을 기억한 채로 대화를 이어갑니다.
* **토큰 관리 및 편의 기능**: 최근 10개의 대화만 유지해 토큰 초과를 방지하며, `!초기화` 명령어로 언제든 대화 기억을 리셋할 수 있습니다.

---

## 2. 사전 준비 및 환경 설정

### 필수 라이브러리 설치
```bash
pip install -U discord.py aiohttp python-dotenv
```

### 환경 변수 설정 (`.env`)
프로젝트 루트 경로에 `.env` 파일을 생성하고 발급받은 토큰과 API 키를 입력합니다.
```env
DISCORD_BOT_TOKEN=여기에_디스코드_봇_토큰_입력
EXAONE_API_KEY=여기에_엑사원_API_키_입력
```

---

## 3. 전체 코드 (`bot.py`)

``` python
import os
import aiohttp
import discord
from discord.ext import commands
from dotenv import load_dotenv

# 1. 환경 변수 로드
load_dotenv()
DISCORD_BOT_TOKEN = os.getenv("DISCORD_BOT_TOKEN")
EXAONE_API_KEY = os.getenv("EXAONE_API_KEY")

EXAONE_URL = "https://exaone.ai"

# 2. Discord 봇 설정 (WebSocket 이벤트 방식)
intents = discord.Intents.default()
intents.message_content = True  # 메시지 본문을 읽기 위한 권한 활성화
bot = commands.Bot(command_prefix="!", intents=intents)

# 채널별 대화 히스토리를 저장할 딕셔너리 (대화 루프 유지용)
chat_histories = {}

def get_history(channel_id: int):
    if channel_id not in chat_histories:
        # 초기 시스템 프롬프트 설정
        chat_histories[channel_id] = [
            {"role": "system", "content": "당신은 친절하고 유능한 AI 어시스턴트입니다."}
        ]
    return chat_histories[channel_id]

# 3. Exaone API 비동기 호출 함수
async def ask_exaone(messages: list) -> str:
    headers = {
        "Authorization": f"Bearer {EXAONE_API_KEY}",
        "Content-Type": "application/json",
    }
    payload = {
        "model": "k-exaone_v2.5",
        "messages": messages,
        "stream": False,
    }

    async with aiohttp.ClientSession() as session:
        async with session.post(EXAONE_URL, headers=headers, json=payload) as resp:
            if resp.status == 200:
                data = await resp.json()
                return data["choices"][0]["message"]["content"]
            else:
                error_text = await resp.text()
                return f"API 오류가 발생했습니다 (상태 코드: {resp.status})\n{error_text}"

# 4. 봇 이벤트 처리
@bot.event
async def on_ready():
    print(f"로그인 완료: {bot.user} (ID: {bot.user.id})")

@bot.event
async def on_message(message: discord.Message):
    # 봇 자신의 메시지 무시 (무한 루프 방지)
    if message.author == bot.user:
        return

    # 명령어(!초기화 등) 우선 처리
    await bot.process_commands(message)

    # 봇이 멘션(@봇이름)되었거나 DM일 때만 대화 루프 실행
    is_mentioned = bot.user in message.mentions
    is_dm = isinstance(message.channel, discord.DMChannel)

    if (is_mentioned or is_dm) and not message.content.startswith("!"):
        # 멘션 태그 제거 후 순수 텍스트 추출
        user_input = message.content.replace(f"<@{bot.user.id}>", "").strip()
        if not user_input:
            return

        # 해당 채널의 대화 내역 불러오기 및 사용자 질문 추가
        history = get_history(message.channel.id)
        history.append({"role": "user", "content": user_input})

        # 최근 10개 메시지(시스템 프롬프트 + 대화 9개)만 유지
        if len(history) > 10:
            history = [history[0]] + history[-9:]
            chat_histories[message.channel.id] = history

        # Discord 채팅창에 '입력 중...' 상태 표시
        async with message.channel.typing():
            reply = await ask_exaone(history)

        # AI 답변을 히스토리에 누적하여 다음 대화에 반영
        history.append({"role": "assistant", "content": reply})

        # Discord 메시지 최대 길이(2,000자) 제한 분할 전송
        if len(reply) > 2000:
            for i in range(0, len(reply), 2000):
                await message.channel.send(reply[i:i+2000])
        else:
            await message.channel.send(reply)

# 5. 대화 내역 초기화 명령어 (!초기화)
@bot.command(name="초기화")
async def reset_chat(ctx: commands.Context):
    chat_histories.pop(ctx.channel.id, None)
    await ctx.send("이 채널의 대화 기억을 초기화했습니다.")

# 6. 봇 실행
if __name__ == "__main__":
    bot.run(DISCORD_BOT_TOKEN)
```

---

## 4. 핵심 로직 설명

1. **멘션 기반 트리거 (`is_mentioned or is_dm`)**
   * 서버의 모든 일반 대화에 봇이 무분별하게 개입하는 것을 막기 위해, `@봇이름`으로 멘션하거나 1:1 DM을 보낼 때만 응답하도록 설계했습니다.
2. **슬라이딩 윈도우 방식의 문맥 관리**
   * 대화가 길어질수록 API 토큰 비용이 증가하고 응답 속도가 느려집니다. 이를 방지하기 위해 맨 앞의 `system` 프롬프트는 고정으로 유지하고, 가장 최근 9개의 대화 내역만 잘라서 API에 전달합니다.
3. **2,000자 제한 분할 처리**
   * Discord는 한 번에 보낼 수 있는 메시지 길이가 최대 2,000자로 제한되어 있습니다. LLM의 긴 답변이 오류 없이 전송되도록 2,000자 단위로 슬라이싱하여 순차 전송합니다.

---

## 5. 실행 및 사용 방법
1. 터미널에서 `python bot.py`를 실행합.
2. 봇이 초대된 Discord 채널에서 `@봇이름 안녕! 오늘 점심 메뉴 추천해 줘`와 같이 말하기.
3. 이어지는 대화에서 `@봇이름 그중에서 매운 걸로 골라줘`라고 물으면 앞선 추천 목록을 기억하고 대답.
4. 새로운 주제로 대화하고 싶을 때는 `!초기화`를 입력해 메모리를 비웁니다.

## 6. 동작화면
<img width="1230" height="696" alt="exaon-llm-test" src="https://github.com/user-attachments/assets/eda5f842-308a-4725-b700-cfc706c12ac6" />
