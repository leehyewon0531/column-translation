## 원문 링크


[Life of a Netflix Partner Engineer — The case of extra 40 ms](https://medium.com/netflix-techblog/life-of-a-netflix-partner-engineer-the-case-of-extra-40-ms-b4c2dd278513)

## 넷플릭스 Partner Engineer의 삶 - 추가적인 40ms의 경우


넷플릭스 앱은 수백대의 스마트 티비, 스트리밍 스틱, 유료 TV 셋톱박스에서 실행된다.넷플릭스 Partner Engineer의 역할은 기기 제조업체가 그들의 기기에서 넷플릭스 앱을 실행하도록 돕는 것이다. 이 글에서 우리는 유럽에 있는 기기에서의 실행을 차단하는 하나의 특히 어려운 이슈에 대해 이야기한다.

**미스테리가 시작된다**

2017년 말쯤, 나는 새로운 셋톱박스에서의 넷플릭스 앱의 문제에 대해 논의하는 전화 회의에 참여했다. 그 박스는 일명 “롤리팝”이라고 불리는, 안드로이드 오픈소스 프로젝트(AOSP) 버전 5.0에 기초한 4k 재생을 지닌 새로운 안드로이드 TV 디바이스였다. 나는 넷플릭스에 몇 년 있던 상태였고, 다수의 기기들을 배송했었지만, 이것은 내 첫 안드로이드 TV 디바이스였다.

장치에 관련된 4명의 플레이어가 모두 회의에 참여했다: 기기를 출시한 유럽의 대형 유료 TV 회사(운영자), 셋톱 박스 펌웨어를 통합한 계약자(통합자), 시스템온칩 제공자(칩 공급자), 그리고 나(넷플릭스).

통합자와 넷플릭스는 이미 엄밀한 넷플릭스 인증 프로세스를 완료한 상태였지만, TV 운영자의 내부 재판 중에 회사에 있는 한 경영진이 심각한 이슈를 신고하였다: 그의 기기의 넷플릭스 재생이 “말을 더듬는다”고. 즉, 비디오가 매우 짧은 시간 재생되었다가, 일시 정지 되었다가, 다시 재생 되었다가, 일시 정지된다는 것. 그것은 항상 일어나는 것은 아니었지만, 확실하게 박스의 전원을 켠 후 며칠 이내에 일어나기 시작했다. 그들은 비디오를 공급했고, 그것은 끔찍하게 보였다.

기기 통합자는 문제를 재현할 방법을 찾았다: 넷플릭스를 반복적으로 시작하고, 재생을 시작하고, 그런 다음 장치 UI로 돌아오는 것. 그들은 프로세스를 자동화하기 위한 스크립트를 제공한다. 때때로 그것은 5분 정도 걸렸지만, 스크립트는 항상 확실하게 버그를 재현했다.

그동안 , 칩 공급업체의 필드 엔지니어는 근본적인 원인을 분석해냈다: 닌자라고 불리는 넷플릭스의 안드로이드 TV 앱은, 충분히 빠른 속도로 오디오 데이터를 전송하지 않고 있었다. 끊김 현상은 장치 오디오 파이프라인의 버퍼 기아 현상으로 인해 발생한 것이었다. 재생은 디코더가 오디오 스트림을 더 전송하기 위해 닌자를 기다리고 있을 때 정지했고, 데이터가 더 도착한 다음에 다시 재생되었다. 통합자와 칩 공급업체, 그리고 운영자 모두 그 문제가 확인되었고, 그들이 내게 주는 메시지는 분명했다: 넷플릭스, 당신의 앱에 버그가 있고, 당신은 그것을 고쳐야 한다. 나는 운영자의 목소리에서 스트레스를 들을 수 있었다. 그들의 기기는 지연되어서 예산을 초과하였고, 그들은 나에게서 결과를 기대했다.

**조사**

나는 의심이 많았다. 같은 닌자 애플리케이션이 스마트 TV들과 다른 셋톱박스들을 포함한 수백만 개의 안드로이드 TV 기기에서 동작한다. 만약 닌자에 버그가 있는 것이라면, 왜 이 기기에만 문제가 일어나는 것일까?

나는 통합자에 의해 제공된 스크립트를 이용하여 문제를 직접 재현하는 것부터 시작했다. 나는 칩 공급업체에 있는 내 상대에게 연락했고, 그에게 전에 이런 것을 한 번도 본 적이 없었냐고 물었다. (그는 본 적이 없다고 했다.) 다음, 나는 닌자의 소스 코드를 읽기 시작했다. 나는 오디오 데이터를 전송할 정밀한 코드를 찾길 원했다. 나는 많은 것을 인지했지만, 재생 코드에서 줄거리(?)를 잃기 시작했고, 나는 도움이 필요했다.

나는 위층으로 올라가서 닌자의 오디오 및 비디오 파이프라인을 쓴 엔지니어를 찾았고, 그는 나에게 코드의 가이드 투어를 주었다. 나는 내 이해를 확인하기 위해 자체 로깅을 추가하며, 소스 코드의 동작 부분을 이해하기 위해 좋은 시간을 보냈다. 넷플릭스 애플리케이션은 복잡하지만, 그것의 가장 단순한 부분에서 그것은 넷플릭스 서버로부터 데이터를 스트리밍하고, 장치에 있는 몇 초 분량의 오디오 및 비디오 데이터를 버퍼링한 다음, 장치의 재생 하드웨어에 오디오 및 비디오 프레임을 한 번에 전송한다.

넷플릭스 애플리케이션에 있는 오디오/비디오 파이프라인을 잠시 이야기해 보자. “디코더 버퍼”까지의 모든 것은 모든 스마트 TV와 셋톱박스에서 동일하지만, A/V 데이터를 기기의 디코더 버퍼로 이동하는 것은 담당 스레드 안에서 동작하는 장치별 루틴이다. 이 루틴의 작업은 오디오 또는 비디오 데이터의 다음 프레임을 제공하는 넷플릭스 제공 API를 호출함으로써 디코더 버퍼를 full로 유지하는 것이다. 닌자에서 이 작업은 안드로이드 스레드에 의해 수행된다. 거기에는 간단한 상태 머신과 다른 재생 상태를 관리하기 위핸 몇몇 로직이 있지만, 일반 재생 아래에서 그 스레드는 데이터의 한 프레임을 안드로이드 재생 API로 복사한 다음, 스레드 스케쥴러에서 15ms를 기다리라고 말하고, 핸들러를 다시 호출한다. 당신이 안드로이드 스레드를 생성할 때, 당신은 스레드가 마치 루프에 빠진 것처럼 반복적으로 실행되도록 요청할 수 있지만, 그것은 당신의 애플리케이션이 아닌 핸들러를 호출하는 안드로이드 스레드 스케줄러이다.

넷플릭스 카탈로그에서 이용 가능한 가장 높은 프레임 속도인 60fps 비디오를 재생하기 위해, 기기는 매 16.66ms마다 새로운 프레임을 렌더링 해야만 하고, 그렇기 때문에 매 15ms마다 새로운 샘플을 확인하는 것은 넷플릭스가 제공하는 모든 비디오 스트림보다 앞서 있을 만큼 빠르다. 왜냐하면 통합자는 오디오 스트림을 문제로서 인식했고, 나는 오디오 샘플을 안드로이드 오디오 서비스에 전송하는 특정한 스레드 핸들러에 집중했다.

나는 이 질문에 답하고 싶다: 추가적인 시간은 어디로 갔는가? 나는 핸들러가 호출한 일부 함수가 범인일 것이라고 가정했고, 문제인 코드가 명백하다고 가정하며, 핸들러 전반에 걸쳐 로그 메시지를 뿌렸다. 곧 밝혀진 것은 핸들러에서 잘못 동작하고 있는 것은 아무것도 없고, 핸들러는 재생이 끊기는 중에도 몇 ms안에 실행되고 있다는 것이었다.

**아하, 통찰**

마지막으로, 나는 3개의 숫자에 집중했다: 데이터 전송 속도, 핸들러가 호출되는 시간, 핸들러가 제어를 안드로이드에게 다시 전달하는 시간. 나는 로그 결과를 파싱하고, 나에게 답을 준 그래프를 만드는 스크립트를 썼다.

![image](https://github.com/leehyewon0531/column-translation/assets/50830078/2913c5fe-7e1d-4a51-a346-390c66db7e8e)


오렌지색 선은 bytes/ms로 데이터를 스트리밍 버퍼로부터 안드로이드 오디오 시스템으로 전송하는 속도이다. 당신은 이 차트에서 3가지 뚜렷한 행동을 볼 수 있다.

1. 데이터 속도가 500 bytes/ms에 도달하는 2개의 길고 성마른 부분. 이 단계는 재생이 시작되기 전에 버퍼링을 하는 중인 단계이다. 핸들러는 데이터를 최대한 빨리 복사하고 있다.
2. 중간에 있는 영역은 일반 재생이다. 오디오 데이터는 약 45 bytes/ms 속도로 이동된다.
3. 끊기는 구역은 오디오 데이터가 거의 10 bytes/ms 속도로 이동하는 오른쪽 부분이다. 이곳은 재생이 유지될 만큼 속도가 빠르지 못하다.

피하기 힘든 결론: 오렌지색 선은 칩 공급업체가 신고한 것을 확실하게 한다. 닌자는 오디오 데이터를 충분히 빠르게 전송하지 않는다는 것을.

이유를 이해하기 위해, 노란색과 회색 선이 말하고 있는 것을 보자.

노란색 선은 핸들러의 위와 아래에서 기록된 타임스탬프로부터 계산된, 핸들러 루틴에서 핸들러가 스스로가 쓰는 시간을 보여준다. 일반 재생 영역과 끊기는 재생 영역 모두에서 핸들러가 쓰는 시간은 약 2ms로 같다. 스파이크는 장치의 다른 작업에 소요된 시간 때문에 런타임이 느려진 인스턴스를 보여준다.

**진짜 근본적인 원인**

핸들러 호출들 사이의 시간인 회색 선은 다른 이야기를 말해준다. 일반 재생의 경우에 당신은 핸들러가 매 15ms마다 한 번씩 호출되는 것을 볼 수 있다. 끊기는 경우, 핸들러는 정확히 매 55ms마다 호출된다. 호출 사이에는 40ms의 여분의 시간이 있고, 재생을 따라갈 수 있는 방법은 없다. 하지만 왜일까?

나는 이 발견을 통합자와 칩 공급업체에 보고했지만(이것 보세요, 안드로이드 스레드 스케쥴러였어요!), 그들은 넷플릭스의 행동에 대해 계속 반발했다. 왜 그냥 핸들러가 호출될 때마다 더 많은 데이터를 복사하지 않느냐? 이것은 공정한 비판이긴 하지만, 이 행동을 변경하는 것은 내가 만드려고 준비한 것보다 더 깊은 변경을 초래하고, 나는 근본적인 원인에 대한 내 연구를 계속했다. 나는 안드로이드 소스 코드에 뛰어들었고, 안드로이드 스레드가 사용자 공간 구성이라는 것을 알게 되었으며, 그 스레드 스케쥴러는 타이밍을 위해 `epoll()` 시스템 콜을 사용한다. 나는 `epoll()` 성능이 보장되지 않음을 알았고, 그래서 나는 어떤 것이 시스템적인 방향으로 `epoll()`에 영향을 끼치고 있다고 의심했다.

이 지점에서, 나는 칩 공급업체에 있는 Marshmallow라고 불리는 안드로이드의 다음 버전에서 이미 고쳐진 버그를 발견한 다른 엔지니어에 의해 보호 받았다. 그 안드로이드 스레드 스케쥴러는 애플리케이션이 포그라운드에서 실행되는지, 백그라운드에서 실행되는지 여부에 따라 스레드의 동작을 변경한다. 백그라운드에 있는 스레드는 추가로 40ms의 대기 시간이 할당된다.

안드로이드 자체의 깊은 버그로 인해, 스레드가 포그라운드로 이동할 때 이 추가 타이머 값이 유지되었다. 대체로 오디오 핸들러 스레드는 애플리케이션이 포그라운드에 있는 동안 생성되었지만, 종종 닌자가 여전히 백그라운드에 있을 때 조금 더 빨리 생성되는 경우도 있었다. 이게 일어나면, 재생은 끊기게 된다.

**배운 교훈**

이것은 우리가 이 플랫폼에서 고친 마지막 버그는 아니지만, 추적하기가 가장 어려운 것이었다. 그것은 넷플릭스 앱의 외부에, 재생 파이프라인의 바깥에 있는 시스템의 부분에 잇는 것이었고, 초기 데이터의 모든 것이 넷플릭스 앱 자체에 있는 버그를 가리키고 있었다.

이 이야기는 실제로 내가 사랑하는 직업의 측면을 예증한다: 나는 파트너가 나에게 던져줄 이슈의 모든 것을 예측할 수 없고, 그것을 고치려면 다수의 시스템을 이해해야 하고, 좋은 동료들과 일해야 하며, 끊임없이 나 스스로를 더 배우라고 부추겨야 한다는 것을 안다. 내가 한 일은 실제 사람들과 좋은 제품에 대한 그들의 즐거움에 직접적인 영향을 미친다. 나는 사람들이 그들의 거실에서 넷플릭스를 즐길 때, 내가 그 일을 만드는 팀의 필수적인 사람이라는 점을 안다.

## 단어장


| 단어 | 의미 |
| --- | --- |
| Partner engineer | 기업이 다른 회사나 조직과 협력할 때 중요한 역할을 하는 직업 |
| pay | 유료 |
| manufacturer | 제조업체 |
| towards | …쪽에, …무렵 |
| conference | 회의, 협의 |
| discuss | 논의하다 |
| playback | 재생 |
| ship | 배송하다 |
| on the call | 통화중 |
| contractor | 계약자 |
| integrate | 통합하다 |
| vendor | 공급업체 |
| rigorous | 엄밀한 |
| certification | 인증 |
| trial | 재판 |
| executive | 경영진 |
| stutter | 말더듬이(본문에서 끊김 현상을 말하는듯) |
| i.e. | 즉 |
| reliably | 확실하게 |
| meanwhile | 그동안, 한편 |
| budget | 예산 |
| skeptical | 의심 많은 |
| reproduce | 낳다, 재생시키다, 재현하다 |
| contact | 연락하다, 접촉하다 |
| counterpart | 짝, 상대 |
| worth | 분량 |
| on-at-a-time | 한 번에 |
| ahead | 앞서 |
| zero in | 총을 쏠 때 가늠자를 맞추다(집중하다, 초점을 맞추다) |
| culprit | 범죄자 |
| sprinkle | 뿌리다 |
| throughout | ~하는 내내, 전반에 걸쳐 |
| apparent | 명백한 |
| misbehave | 나쁜 행동을 하다 |
| distinct | 별개의, 뚜렷한 |
| spiky | 대못 같은, 성마른 |
| due to | ~때문에 |
| push back | 반발하다, 밀어내다 |
| criticism | 비판 |
| construct | 구성 |
| plumb | 추, 수직인, 수직인지 아닌지를 조사하다, 수직이 되게 하다 |
| exemplify | 예시하다, 예증하다 |
