---
title: "디스코드 봇에 실시간 날씨 알림 통합하기"
date: 2026-04-19 14:00:00 +0900
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
 - discord.js
 - typescript
 - weather-api
 - automation
categories:
 - 습작만들기
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---


Gemini와 Docker를 통합한 디스코드 봇 프로젝트를 진행 중.
그중 실용적인 기능인 **실시간 날씨 알림** 시스템을 추가함.
기상청(KMA) 오픈 API를 사용하여 구현한 방법을 공유함.

## 1. 개요

봇이 매시간 특정 디스코드 채널에 현재 날씨 정보를 자동으로 전송하는 것이 목표.

### 주요 파일:

- `work/docker_prep/discord_echo_bot/discord-bot-gemini.ts`: 날씨 조회 및 알림 스케줄링 로직.
- `.env`: `DISCORD_TOKEN` 및 `KMA_SERVICE_KEY` 등 민감 정보 저장.

## 2. 구현 상세

### 기상청 날씨 데이터 가져오기

[data.go.kr](https://www.data.go.kr/)의 `VilageFcstInfoService_2.0`(단기예보) API를 사용함. API가 특정 시간대(0200, 0500, 0800 등)에 데이터를 제공하므로 현재 시간에 맞춰 `baseTime`을 계산하는 로직이 포함됨.

```typescript
// 날씨 데이터 인터페이스
interface WeatherData {
  temp: string;
  skyText: string;
}

// 기상청 날씨 조회 함수
async function getKMAWeather(): Promise<WeatherData | null> {
  const serviceKey = process.env.KMA_SERVICE_KEY;
  const url = 'http://apis.data.go.kr/1360000/VilageFcstInfoService_2.0/getVilageFcst';

  const now = new Date();
  const baseDate = now.toISOString().slice(0, 10).replace(/-/g, '');

  // baseTime 계산 로직...
  // (0200, 0500, 0800, 1100, 1400, 1700, 2000, 2300)

  const nx = '62'; // 지역 좌표 (예: 서울/OO고 인근)
  const ny = '122';

  try {
    const response = await axios.get(url, {
      params: {
        serviceKey: serviceKey,
        pageNo: 1,
        numOfRows: 200,
        dataType: 'JSON',
        base_date: baseDate,
        base_time: baseTime,
        nx: nx,
        ny: ny,
      },
    });

    const items = response.data?.response?.body?.items?.item;
    const temp = items.find((i: any) => i.category === 'TMP')?.fcstValue;
    const sky = items.find((i: any) => i.category === 'SKY')?.fcstValue;

    const skyText = sky === '1' ? '☀️ 맑음' : sky === '3' ? '☁️ 구름많음' : '☁️ 흐림';
    return { temp, skyText };
  } catch (error) {
    console.error(`날씨 API 에러: ${error.message}`);
    return null;
  }
}
```

### node-cron을 이용한 자동화

`node-cron`을 사용하여 작업을 스케줄링함.

```typescript
client.once(Events.ClientReady, (c) => {
  console.log(`✅ ${c.user?.tag}로 로그인 성공`);

  // 60분마다 실행
  cron.schedule('*/60 * * * *', async () => {
    if (!CHANNEL_ID) return;
    const channel = await c.channels.fetch(CHANNEL_ID);
    const weather = await getKMAWeather();

    if (channel instanceof TextChannel) {
      if (weather) {
        await channel.send(`🌡️ **현재 날씨 알림**\n- 기온: ${weather.temp}°C\n- 상태: ${weather.skyText}\n(60분 주기 자동 알림)`);
      }
    }
  });
});
```

## 3. Docker 배포

- 이 프로젝트의 핵심은 모든 기능을 Docker 컨테이너 내에서 실행.
- 봇은 백그라운드에서 날씨 알림 주기를 유지하면서, 복잡한 명령 처리를 위해 `gemini-cli` 이미지를 호출할 수 있음.

## 4. 결론

기상청 날씨 API와 같은 실제 데이터를 통합하여 디스코드 봇의 활용도를 높임.


## Screen shot 1

<img width="684" height="86" alt="Image" src="https://github.com/user-attachments/assets/af809208-fce1-41cc-8a1f-17cbd29803ba" />

## Screen shot 2

<img width="431" height="535" alt="Image" src="https://github.com/user-attachments/assets/39a28cd7-43c3-491d-8387-1abf33e6d388" />

