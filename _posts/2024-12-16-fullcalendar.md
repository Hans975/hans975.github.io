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



# Method

```react
const [checkIn, setCheckIn] = useState(null);
const [checkOut, setCheckOut] = useState(null);
const [highlightedRange, setHighlightedRange] = useState([]);
```

체크인 날짜, 체크아웃 날짜, 선택된 날짜 범위의 상태를 관리한다.



## handleDateClick

```react
const handleDateClick = (info) => {}
```

사용자가 날짜를 클릭하면 실행된다. 이번 코드에서는 클릭한 날짜에 따라 체크인과 체크아웃 날짜를 설정하거나 수정하는 역할을 한다.



### 체크인 날짜 설정

```react
if(!checkIn){
	setCheckIn(info.dateStr);
	setCheckout(null);
	setHilightedRange([clickedDate]);
	console.log('체크인 설정:', info.dateStr);
}
```

* 처음 날짜를 클릭했을 때 checkIn이 null이면 체크인 날짜로 선택하고,  체크아웃 날짜를 초기화한다. 이때 선택된 체크인 날짜를 강조하기 위해 hilightedRange에 추가한다.



### 체크아웃 날짜 설정

```react
else if(!checkOut){
	if(clickDate < new Date(checkIn)){
        setChekIn(info.dateStr);
        setCheckOut(null);
        setHighlightedRange([clickedDate]);
        console.log('체크인 날짜를 새로 설정:', info.dateStr);
    } else {
        setCheckOut(info.dateStr);
        
        const range = [];
        let currentDate = new Date(checkIn);
        const endDate = new Date(info.dateStr);
        while(currentDate <= endDate){
            range.push(new Date(currentDate));
            currentDate.setDate(currentDate.getDate()+1);
        }
        setHighlightedRange(range);
        console.log('체크인:', checkIn, '체크아웃:', info.dateStr);
    }
}
```

* 체크인 날짜가 설정되어 있고, 체크아웃 날짜가 없다면 클릭한 날짜가 체크인 날짜 이후일 경우 체크아웃 날짜로 설정한다. 이때 선택된 날자 범위를 highlightedRange에 저장한다.
* 만약 클릭한 날짜가 체크인 날짜보다 이전 날짜라면, 체크인 날짜를 새로 설정하고 체크아웃 날짜를 초기화하여 다시 시작한다.

### 체크인 또는 체크아웃 날짜 수정

```react
else {
	if(clickeDate < new Date(checkIn)){
        setCheckIn(info.dateStr);
        setCheckOut(null);
        setHighlightedRange([clickedDate]);
        console.log('체크인 날짜 수정:', info.dateStr);
    } else if(clickedDate > new Date(checkIn)){
        setCheckOut(info.dateStr);
        
        const range = [];
        let currentDate = new Date(checkIn);
        const endDate = new Date(info.dateStr);
        while(currentDate <= endDate) {
            range.push(new Date(currentDate));
            currentDate.setDate(currentDate.getDate()+1);
        }
        setHighlightedRange(range);
        console.log('체크인:', checkIn, '체크아웃:', info.dateStr);
    }
}
```

* 체크인과 체크아웃 날짜가 모두 설정된 상태에서 날짜를 클릭하면, 날짜 범위를 재설정하거나 수정할 수 있다. 클릭한 날짜가 체크인 날짜보다 이전이면 체크인 날짜를 수정하고, 체크아웃 날짜는 수정하지 않는다. 반대로 체크아웃 날짜가 클릭된 경우 체크아웃 날짜를 수정하여 날짜 범위를 다시 계산한다.

```react
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
```

* 범위 내에 이미 체크아웃 날짜가 있으면, 체크아웃 날짜만 수정하고 범위를 다시 계산한다.

## isHighlighted

```react
// 날짜가 선택된 범위에 포함되는지 확인
const isHighlighted = (date) => {
	return highlightedRange.some((d) => d.toDateString() === date.toDateString());
};
```

이 메서드는 주어진 날짜가 highlightedRange 배열에 포함되어 있는지 확인하는 함수이다. highlightedRange 배역에 날짜가 포함되면 true를 반환하고, 그렇지 않으면 false를 반환한다.

* highlightedRange는 선택된 체크인과 체크아웃 날짜 사이의 모든 날짜를 저장하고 있으므로, 이 함수는 주어진 날짜가 해당 범위에 포함되는지 여부를 확인한다.
* 날짜가 범위에 포함되면 해당 날짜에 스타일 클래스를 적용하기 위해 highlighted 클래스를 반환한다.

## dayCellClassNames

```react
// 날짜 범위에 클래스를 추가
dayCellClassNames={(info) => {
  if (isHighlighted(info.date)) {
    return 'highlighted'; // 날짜가 선택된 범위에 포함되면 'highlighted' 클래스 추가
  }
  return ''; // 기본 클래스는 빈 문자열
}}
```

이 메서드는 fullcalendar에서 각 날짜에 대해 클래스 이름을 동적으로 지정하는 함수이다. dayCellClassNames는 각 날짜 셀에 대해 길행되며, 날짜가 highlightedRange에 포함되면 highlighted 클래스를 반환하여 해당 날짜 셀에 스타일을 적용한다.

* isHighlighted 메서드를 사용하여 날짜가 선택된 범위에 포함되는지 확인하고, 포함되면 highlighted 클래스를 추가한다. 이 클래스는 css 스타일에서 날짜의 배경색을 변경하는 데 사용된다.

## FullCalendar 컴포넌트

```react
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
```

fullcalendar는 달력을 렌더링하는 컴포넌트로, 다양한 옵션을 설정하여 달력의 동작을 정의한다. 여기서 사용된 주요 속성들은 다음과 같다.

* plugins: 사용하려는 플러그인 설정
  - dayGridPlugin: 월간 달력 뷰를 제공
  - interactionPlugin: 날짜 선택 가능
  - timeGridPlugin: 시간 기반의 일정 뷰를 제공
* initialView: 달력의 초기 뷰를 설정
  - dayGridMonth: 월별 달력을 보여준다.
* selectable: 날짜 선택 가능
* dateClick: 날짜 클릭 시 실행할 핸들러 함수. 여기서는 handleDateClick을 지정하여 날짜 클릭 시 날짜를 선택하고 범위를 계산한다.
* headerToolbar: 달력 상단의 도구 모음 버튼 설정. prev, next, today, title 등이 포함된다.
* validRange: 달력에서 선택할 수 있는 날짜 범위를 설정. 여기서는 오늘 날짜부터 시작하도록 설정