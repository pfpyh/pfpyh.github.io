---
layout: post
title:  "[칼만필터] 칼만 필터 알고리즘"
date:   2022-08-15 09:00:00 +0900
categories: Other KalmanFilter
use_math: true
---
센서 융합을 위해 칼만 필터를 구현해보고 최종적으로 아두이노를 활용해서 물체의 자세를 계산해보고자 합니다.
두 번째 이야기로 칼만 필터 알고리즘에 대해 간략하게 알아보고 C++로 구현해보았습니다.
예시를 위한 데이터와 구현은 "칼만 필터는 어렵지 않아"라는 책을 참조했습니다.
  
### Algorithm
칼만 필터는 크게 예측(Predict)과 추정(Estimate)로 이루어진다.
  
|기호|이름|설명|
|----|----|----|
|\\(\hat{a}_{t}\\)|추정값|hat이 붙으면 추정값이라는 의미
|\\(\hat{a}_{\bar{t}}\\)|예측값|hat과 아랫 첨자에 bar가 있으면 예측값이라는 의미|
  
**예측값 계산**  
  
$$
\begin{align}
&\hat{x}_{\bar{t}}=A\hat{x}_{t-1}\\
&P_{\bar{t}}=AP_{t-1}A^{T}+Q\\
\end{align}
$$
  
**Kalman Gain 계산**  
  
$$
\begin{align}
K_{t}=P_{\bar{t}}H^{T}(HP_{\bar{t}}H^{T}+R)^{-1}\\
\end{align}
$$
  
**추정값 계산**  
  
$$
\begin{align}
\hat{x}_t=\hat{x}_{\bar{t}}+K_{t}(z_{t}-H\hat{x}_{\bar{t}})\\
\end{align}
$$
  
**오차 공분산 계산**  
  
$$
\begin{align}
P_{t}=P_{\bar{k}}-K_{t}HP_{\bar{t}}\\
\end{align}
$$
  
|기호|이름|설명|
|----|----|----|
|\\(x_{t}\\)|상태변수|(n x 1) 내부 계산용|
|\\(z_{t}\\)|측정값|(m x 1) 센서로부터 입력된 값|
|\\(A\\)|시스템 행렬|별도로 설명|
|\\(H\\)|출력 행렬|(m x n) 입력 중 사용할 값을 의미|
|\\(w_{t}\\)|시스템 잡음|(n x 1) 상태 변수에 영향을 주는 잡음|
|\\(Q\\)||\\(w_{t}\\)의 공분산 행렬|
|\\(v_{t}\\)|측정 잡음|(m x 1) 센서에서 유입되는 잡음|
|\\(R\\)||\\(v_{t}\\)의 공분산 행렬|
|\\(P\\)|오차공분산|별도로 설명|
|\\(K\\)|Kalman Gain|별도로 설명|
  
### 예측(Predict)
이전 추정값을 이용해서 현재 시점의 예측값 \\(\hat{x}_{\bar{t}}\\) 을 구하는 과정이다. 이때 예측을 위해 선형식 \\(A\\)가 필요하다. \\(A\\)는 구하고자 하는 값을 시스템 모델링을 통해 나타낸 것으로 **시스템 행렬** 이라고 하고 칼만 필터에서 가장 중요한 부분 중 하나이다.
  
$$
\begin{aligned}
&\hat{x}_{\bar{t}}=A\hat{x}_{t-1}\\
\end{aligned}
$$
  
그리고 오차 공분산 또한 예측값을 구하게 되는데 오차 공분산에 대해서는 뒤에 별도로 정리하겠다.
  
$$
\begin{aligned}
&P_{\bar{t}}=AP_{t-1}A^{T}+Q
\end{aligned}
$$
  
### 추정(Estimate)
예측값을 이용해서 Kalman Gain)을 계산하고 이를 사용해서 추정값을 구한다.
  
Kalman Gain은 일종의 보정값으로 예측값과 측정값 \\(z_{t}\\) 사이에서 가중치를 계산하는 것을 말한다.
  
$$
\begin{aligned}
K_{t}=P_{\bar{t}}H^{T}(HP_{\bar{t}}H^{T}+R)^{-1}
\end{aligned}
$$
  
추정값 계산은 Kalman gain을 통해 예측값과 측정값 사이의 비율을 조정해서 계산한다.
  
$$
\begin{aligned}
\hat{x}_t=\hat{x}_{\bar{t}}+K_{t}(z_{t}-H\hat{x}_{\bar{t}})
\end{aligned}
$$
  
위 식은 전개하여 \\(H\\)를 identity로 치환해서 보면 LPF와 닮아 있음을 알 수 있다.
  
$$
\begin{aligned}
\hat{x}_t&=\hat{x}_{\bar{t}}+K_{t}(z_{t}-H\hat{x}_{\bar{t}})\\
&=\hat{x}_{\bar{t}}+K_{t}z_{t}-K_{t}H\hat{x}_{\bar{t}}\\
&=(I-K_{t}H)\hat{x}_{\bar{t}}+K_{t}z_{t}\quad\quad\text{H에 I를 대입}\\
&=(I-K_{t}I)\hat{x}_{\bar{t}}+K_{t}z_{t}\\
&=(I-K_{t})\hat{x}_{\bar{t}}+K_{t}z_{t}\\
\end{aligned}
$$
  
$$
\begin{aligned}
\bar{x}_{t}&=\alpha\bar{x}_{t-1}+(1-\alpha)x_{t}\quad\quad\text{ a에 (1-K)를 대입}\\
&=(1-K)\bar{x}_{t-1}+Kx_{t}
\end{aligned}
$$
  
따라서 칼만 필터는 직전 추정값에 적절한 가중치를 곱하여 사용하는 LPF와 유사하게 예측값과 측정값 사이에 가중치(Kalman gain)을 곱하여 사용하는 것이다.
  
### 오차 공분산 (Error Covariance)
> 여기서는 칼만 필터에서의 역할만 알아보고 상세한 내용은 추후 별도의 글을 통해 알아보겠습니다
  
칼만 필터의 예측, 추정 과정은 정규 분포를 따른다. 따라서 오차 공분산은 해당 값들이 참값으로부터 얼마나 떨어져있는지를 나타내기 때문에 정확도의 척도로 사용이 가능하다. 
  
### 구현
구현은 굉장히 단순하다. 위 알고리즘을 기반으로 예측, 추정 단계를 구현하고 추정값을 반환하는 형태로 구성했다. 이때 시스템 변수들은 객체를 생성하는 단계에서 입력하도록 하였다.
  
시스템 변수들은 이후 구현할 IMU 칼만 필터를 통해 정의하고 구현을 추가할 예정이다.
```cpp  
template <typename T>  
class KalmanFilter
{
private :
    T _H;
    T _Q;
    T _R;
    T _x;
    T _P;
    bool _first = true;
  
public :
    KalmanFilter(const T& H,
                 const T& Q,
                 const T& R,
                 const T& x,
                 const T& P)
        : _H(H), _Q(Q), _R(R), _x(x), _P(P) {};
  
    auto run(const T& A, const T& z) -> const decltype(_x)&
    {
        if (!_first)
        {
            // predict
            auto xp = A * _x;
            auto Pp = A * _P * A.transpose() + _Q;
  
            // kalman gain
            auto K = (Pp * _H.transpose()) * util::inverse(_H * Pp * _H.transpose() + _R);
  
            // estimate
            _x = xp + K * (z - (_H * xp));
            _P = Pp - K * _H * Pp;
        }
        else _first = false;
        return _x;
    };
};
```  
----