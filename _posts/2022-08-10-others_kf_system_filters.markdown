---
layout: post
title:  "[칼만필터] 시스템과 필터"
date:   2022-08-11 09:00:00 +0900
categories: Other KalmanFilter
use_math: true
---
센서 융합을 위해 칼만 필터를 구현해보고 최종적으로 아두이노를 활용해서 물체의 자세를 계산해보고자 합니다.
첫 번째 이야기로 칼만 필터를 이해하기 위해 필요한 기본적인 배경 지식을 정리해보았습니다.
예시를 위한 데이터와 구현은 "칼만 필터는 어렵지 않아"라는 책을 참조했습니다.
  
## System
> 각 구성요소들이 상호작용하거나 상호의존하여 복잡하게 얽힌 통일된 하나의 집합체(unified whole)다. 또는 이 용어는 복잡한 사회적 체계의 맥락에서 구조와 행동을 통제하는 규칙들의 집합체를 일컫기도 한다.  
Ref. <https://ko.wikipedia.org/wiki/%EC%8B%9C%EC%8A%A4%ED%85%9C>
  
센서 값을 받아 어떤 데이터로 가공한다고 하자. 그러기 위해서는 센서 자체의 특성, 오차, 화이트 노이즈 등 다양한 요소를 고려해야 한다. 이러한 여러 요소를 모두 묶어서 하나의 시스템이라고 볼 수 있다.
  
## 제어 공학
> 제어 이론에 기반하여 동적 시스템의 동작이 원하는 대로 이루어지도록 하는 방법을 연구하는 공학의 한 분야.  
Ref. <https://namu.wiki/w/%EC%A0%9C%EC%96%B4%EA%B3%B5%ED%95%99>
  
입력을 통해 원하는 출력이 나오도록 입력을 모델링 / 근사화하여 시스템을 구성하고 이를 제어할 수 있도록 설계하는 과정을 말한다.
  
실제 CPU와 같은 하드웨어로 구성된 시스템도 있지만 그와 유사한 역할을 수행하는 소프트웨어 모듈로 구성된 것도 포함한다.
- 이 경우에는 각 모듈의 모음을 전달 함수라고 한다.
  
## Filter
잡음이 포함된 신호를 걸러내는 과정을 필터라고 하는데 주로 재귀 필터를 의미한다.
  
재귀 필터란 현재 입력값과 이전 결과값을 특정한 수식을 통해 계산하여 현재 결과값을 도출하는 형태의 필터를 말한다.
  
재귀 필터를 사용하는 이유는 이전 결과값만 저장하고 있으면 되기 때문에 저장 공간 면에서 효율적이고 연산 또한 짧기 때문이다.
  
필터를 구성하기 위해서는 먼저 입력 값에 대한 모델링이 이루어져야 하고 잡음 요소 등을 제거하기 위한 값이나 수식이 있어야 한다. 
  
### Example: 1차 Low-Pass filter
Low-Pass filter(이하 LPF)는 저주파 신호는 통과시키고 고주파 신호는 걸러내는 형태의 필터를 말한다. 잡음 요소는 대부분 고주파 성분으로 되어 있기 때문에 신호를 처리하는데 많이 사용되는 필터이다.
  
예시로는 책에서 제공하는 1차 LPF를 사용하겠다.
  
$$ 
\begin{aligned} 
\bar{x}_{t}=\alpha\bar{x}_{t-1}+(1-\alpha){x}_{t}\\
\end{aligned} 
$$  
  
$$
\begin{aligned} 
&\bar{x}_{t}: 현재 추정값\\
&x_{t}: 현재 측정값\\
&\alpha: 상수 (0 < \alpha < 1)\\
\end{aligned} 
$$  
  
위 식에 의해 t-1에 대한 식은 아래와 같이 된다.  
  
$$ 
\begin{aligned} 
\bar{x}_{t-1}=\alpha\bar{x}_{t-2}+(1-\alpha)x_{t-1}\\ 
\end{aligned} 
$$  
  
위 식을 다시 t에 대입해서 정리하면
  
$$
\begin{aligned} 
\bar{x}&=\alpha\bar{x}_{t-1}+(1-\alpha)x_{t}\\
&=\alpha(\alpha\bar{x}_{t-2}+(1-\alpha)x_{t-1})+(1-\alpha)x_{t}\\
&=\alpha^2\bar{x}_{t-2}+\alpha(-\alpha)x_{t-1}+(1-\alpha)x_{t}\\
\end{aligned}
$$
  
위와 유사하게 t-2에 대해서 전개해서 t에 대입하면
  
$$
\begin{aligned}
\bar{x}&=\alpha^3\bar{x}_{t-3}+\alpha^2(1-\alpha)x_{t-2}+\alpha(1-\alpha)x_{t-1}+(1-\alpha)x_{t}\\
\end{aligned}
$$
  
와 같이 정리가 가능하다. 여기서 \\(0 < \alpha < 1\\) 이기 때문에
  
$$
\begin{aligned}
\alpha^2(1-\alpha) < \alpha(1-\alpha) < 1-\alpha\\
\end{aligned}
$$
  
가 성립한다. 따라서 *1차 LPF는 이전 측정 값일수록 더 작은 가중치를 받는 다는 것을 알 수 있다.*
  
```cpp  
// LPF에 대한 구현
template <typename T>
class LowPassFilter
{
private:
    T _prev;
    T _alpha ;
    bool _first = true;
  
public:
    LowPassFilter(const T& init, const T& alpha)
        : _prev(init), _alpha(alpha) {};
  
    auto run(const T& x)->T
    {
        if (_first)
        {
            _prev = x;
            _first = false;
        }
        _prev = _alpha * _prev + (1 - _alpha) * x;
        return _prev;
    };
};
```  
  
```cpp
// 사용 예제
...
constexpr char SONAR_ALT_CSV[] = "SonarAlt.csv";
constexpr char OUTPUT_1st_SONAR_ALT_CSV[] = "Output_1st_SonarAlt.csv";
constexpr char OUTPUT_2nd_SONAR_ALT_CSV[] = "Output_2nd_SonarAlt.csv";
...
constexpr double x0 = 0.0;
constexpr double alpha = 0.4;
filters::LowPassFilter<double> lpf(x0, alpha);
  
constexpr uint32_t sample_count = 500;
CSV csv(SONAR_ALT_CSV);
auto sonar_map = csv.read(',');
  
csv.open(OUTPUT_1st_SONAR_ALT_CSV, true);
std::vector<std::string> write_buf;
write_buf.push_back(std::string());
for (uint32_t i = 0; i < sample_count; ++i)
{
    auto sonar = std::stod(sonar_map[i][0]);
    auto filtered_sonar = lpf.run(sonar);
    write_buf[0] = std::to_string(filtered_sonar);
    csv.write(write_buf);
}
...
```  
  
***