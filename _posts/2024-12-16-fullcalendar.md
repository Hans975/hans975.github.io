---

layout: single
title: "달력 만들기-fullcalendar"
toc: true
typora-root-url: ../

---

# 최종 목표

![스크린샷 2024-12-16 164340](/../../../Pictures/Screenshots/스크린샷 2024-12-16 164340.png)

1. 달력 생성
2. 첫번째 날짜는 체크인으로 인식되고 두번째 날짜는 체크아웃으로 인식
3. 체크인 이후의 날짜를 클릭하면 체크아웃 날짜로 입력
4. 체크인 이전의 날짜를 클릭하면 전체 초기화 후 그 날짜를 체크인으로 입력

```react
import React, { useState } from 'react';
import FullCalendar from '@fullcalendar/react';
import dayGridPlugin from '@fullcalendar/daygrid';
import interactionPlugin from '@fullcalendar/interaction'; // for selectable
import timeGridPlugin from '@fullcalendar/timegrid'; // for timeGrid views

export default function Calendar() {
  const [checkIn, setCheckIn] = useState(null);
  const [checkOut, setCheckOut] = useState(null);
  const [highlightedRange, setHighlightedRange] = useState([]);
  
  // 날짜 클릭 시 처리
  const handleDateClick = (info) => {
    const clickedDate = new Date(info.dateStr);

    // 첫 번째 클릭: 체크인 날짜 설정
    if (!checkIn) {
      setCheckIn(info.dateStr);
      setCheckOut(null);  // 체크인만 설정되고 체크아웃은 초기화
      setHighlightedRange([clickedDate]);  // 선택된 체크인 날짜를 강조
      console.log('체크인 설정:', info.dateStr);
    } 
    // 두 번째 클릭: 체크아웃 날짜 설정 또는 수정
    else if (!checkOut) {
      if (clickedDate < new Date(checkIn)) {
        // 체크인 날짜보다 이전 날짜를 클릭한 경우: 구간 초기화 후 다시 시작
        setCheckIn(info.dateStr);
        setCheckOut(null);  // 체크아웃 초기화
        setHighlightedRange([clickedDate]);  // 새로운 체크인 날짜 강조
        console.log('체크인 날짜를 새로 설정:', info.dateStr);
      } else {
        // 체크인 날짜 이후의 날짜를 클릭한 경우: 체크아웃 날짜만 업데이트
        setCheckOut(info.dateStr);

        // 선택된 날짜 범위 저장
        const range = [];
        let currentDate = new Date(checkIn);
        const endDate = new Date(info.dateStr);
        while (currentDate <= endDate) {
          range.push(new Date(currentDate));
          currentDate.setDate(currentDate.getDate() + 1);
        }
        setHighlightedRange(range);
        console.log('체크인:', checkIn, '체크아웃:', info.dateStr);
      }
    }
    // 체크인과 체크아웃이 모두 설정된 후의 클릭: 체크인 또는 체크아웃 수정
    else {
      if (clickedDate < new Date(checkIn)) {
        // 새로운 체크인 날짜로 설정하고 체크아웃 날짜 초기화
        setCheckIn(info.dateStr);
        setCheckOut(null);
        setHighlightedRange([clickedDate]);
        console.log('체크인 날짜 수정:', info.dateStr);
      } else if (clickedDate > new Date(checkOut)) {
        // 체크아웃 날짜 수정
        setCheckOut(info.dateStr);

        // 선택된 날짜 범위 저장
        const range = [];
        let currentDate = new Date(checkIn);
        const endDate = new Date(info.dateStr);
        while (currentDate <= endDate) {
          range.push(new Date(currentDate));
          currentDate.setDate(currentDate.getDate() + 1);
        }
        setHighlightedRange(range);
        console.log('체크인:', checkIn, '체크아웃:', info.dateStr);
      }
      // 체크아웃 날짜가 이미 구간 내에 있는 경우: 구간 재설정
      else if (clickedDate >= new Date(checkIn) && clickedDate <= new Date(checkOut)) {
        setCheckOut(info.dateStr);

        // 구간을 다시 계산하여 업데이트
        const range = [];
        let currentDate = new Date(checkIn);
        const endDate = new Date(info.dateStr);
        while (currentDate <= endDate) {
          range.push(new Date(currentDate));
          currentDate.setDate(currentDate.getDate() + 1);
        }
        setHighlightedRange(range);
        console.log('체크인:', checkIn, '체크아웃(수정):', info.dateStr);
      }
    }
  };

  // 날짜가 선택된 범위에 포함되는지 확인
  const isHighlighted = (date) => {
    return highlightedRange.some((d) => d.toDateString() === date.toDateString());
  };

  return (
    <div style={{ maxWidth: '800px', margin: '0 auto'}}>
      <style>
        {`
          .highlighted {
            background-color: #d3d3d3; /* 회색 배경색 */
            color: black; /* 텍스트 색상 */
          }
        `}
      </style>
      <FullCalendar
        plugins={[dayGridPlugin, interactionPlugin, timeGridPlugin]}
        initialView="dayGridMonth"
        selectable={true}  // 날짜 선택 가능
        dateClick={handleDateClick}  // 날짜 클릭 시 처리
        headerToolbar={{
          left: 'prev,next today',
          center: 'title',
          right: 'dayGridMonth,ti meGridWeek,timeGridDay'
        }}
        validRange={{
          start: new Date() //오늘 날짜부터 시작
        }}
        // 날짜 범위에 클래스를 추가
        dayCellClassNames={(info) => {
          if (isHighlighted(info.date)) {
            return 'highlighted'; // 날짜가 선택된 범위에 포함되면 'highlighted' 클래스 추가
          }
          return ''; // 기본 클래스는 빈 문자열
        }}
      />
      {checkIn && checkOut && (
        <div>
          <p>체크인: {checkIn}</p>
          <p>체크아웃: {checkOut}</p>
        </div>
      )}
    </div>
  );
}

```





# fullcalendar 사용법

설치:

```cmd
npm install @fullcalendar/react@latest 
```

: fullcalendar의 react용 패키지 설치

```cmd
npm install @fullcalendar/core@latest 
```

: fullcalendar 기본 기능을 제공하는 core 패키지 설치

```cmd
npm install @fullcalendar/daygrid@latest 
```

: daygrid view(날짜를 그리드 형식으로 제표시하는 뷰)를 제공하는 fullcalendar 패키지 설치

```cmd
npm install @fullcalendar/interaction 
```

: fullcalendar 상의 이벤트를 드래그 앤 드롭하거나, 클릭 등의 상호작용을 처리하는 기능을 제공하는 기능을 제공하는 패키지 설치

```cmd
npm install @fullcalendar/timegrid 
```

: 시간 기반의 일정을 표시하는 timegrid 뷰를 제공하는 패키지 설치

```cmd
npm install web-vitals 
```

: 웹 애플리케이션의 핵심 성능 지표(로딩 시간, 반응 속도 등)을 측정하고 보고하는 데 사용하는 라이브러리 설치