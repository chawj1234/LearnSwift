#### EKEventStore: 캘린더 데이터베이스의 총괄 매니저
##### 주요 프로퍼티
- sources: 캘린더 계정 소스(iCloud, Goolge등) 목록을 가져온다.
- calendars(for: ): 특정 타입(이벤트 또는 미리 알림)의 모든 캘린더 목록을 가져온다.
- defaultCalendarForNewEvents: 새로운 이벤트를 저장할 때 사용되는 기본 캘린더 알려준다.
##### 핵심 메서드
- requestFullAccessToEvents(completion: ): 사용자에게 캘린더 전체 접근 권한을 요청한다. (iOS 17+)
- requestWriteOnlyAccessToEvents(completion: ): 사용자에게 쓰기 전용 권한을 요청한다. (iOS 17+)
- save(\_:span:commit: ): 이벤트나 미리 알림을 데이터베이스에 저장한다. `span`파라미터는 반복 이벤트일 경우, 이번 이벤트만 수정할지(`thisEvent`) 아니면 향후 모든 이벤트를 수정할지(`futureEvents`) 결정한다.
- remove(\_:span:commit: ): 이벤트나 미리 알림을 삭제한다.
- events(matching: ): 특정 기간에 해당하는 모든 이벤트를 가져온다(fetch).
- predicateForEvents(withStart:end:calendars: ):`events(matching: )`메서드에서 사용할 검색 조건(predicate)을 만듭니다.

#### EKEvent: 특정 이벤트의 제목, 시작종료, 위치
##### 주요 프로퍼티
- title: 이벤트의 제목
- startDate / endDate: 이벤트의 시작 및 종료 시간
- isAllDay: true로 설정하면 하루 종일 이벤트
- location: 이벤트가 열리는 장소
- notes: 이벤트에 대한 메모나 설명
- url: 이벤트와 관련된 웹 페이지 주소
- calendar: 이 이벤트가 속해있는 EKCalendar 객체
- attendees: 이벤트 참석자 목록 (\[EKParticipant])
- organizer: 이벤트를 주최한 사람
- alarms: 이벤트 시작 전에 알림을 설정합니다 (\[EKAlarm]). 예를 들어 "15분 전 알림"을 추가할 수 있다.
- recurrenceRules: 반복 규칙을 설정합니다 (\[EKRecurrenceRule]). "매주 화요일 반복" 같은 규칙을 만들 수 있다.

#### EKReminder: 미리 알림 앱의 할 일 항목
##### 주요 프로퍼티
- title: 할 일의 제목
- notes: 할 일에 대한 메모
- isCompleted: 할 일의 완료 여부 (`true` 또는 `false`).
- completionDate: 할 일을 완료한 날짜
- dueDateComponents: 마감일을 `DateComponents`로 설정한다. 시간까지 지정할 수 있다.
- startDateComponents: 할 일의 시작일을 설정한다.
- priority: 할 일의 우선순위를 설정한다. (1~9 사이, 1이 가장 높음)

#### EKCalendar(캘린더)
- title: 캘린더의 이름
- source: 캘린더가 속한 계정 소스(`EKSource`)
- allowsContentModifications: 캘린더의 내용을 수정할 수 있는지 여부
- cgColor: 캘린더의 대표 색상
#### EKParticipant(참석자)
- name: 참석자의 이름
- participantRole: 참석자의 역할(필수, 선택 등)
- participantStatus: 참석자의 응답 상태(수락, 거절, 미정 등)

#### EKRecurrenceRule (반복 규칙)
- init(recurrenceWith:interval:end:) init을 통해 반복 규칙을 생성
- frequency: 반복 주기(`daily`, `weekly`, `monthly`, `yearly`)를 설정
- interval: `frequency`를 몇 번마다 반복할지 설정 (예: `frequency`가 `weekly`이고 `interval`이 2이면 2주마다 반복)
- daysOfTheWeek: 주간 반복 시 특정 요일(`EKRecurrenceDayOfWeek`)을 지정