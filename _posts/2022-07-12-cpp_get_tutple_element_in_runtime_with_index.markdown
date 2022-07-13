---
layout: post
title:  "std::tuple의 element를 index를 통해 runtime에 접근하기"
date:   2022-07-12 10:00:00 +0900
categories: C++
---
Database를 통해 읽어온 data를 처리하기 위한 container를 생성한다고 할 때, 보통은 type이 고정되어 있기 때문에 객체를 일일이 생성해줘도 되지만 좀 더 generic하게 coding하기 위해 tuple을 사용하기로 결정했다.  
std::tuple은 훌륭한 generic container이지만 runtime에 index를 통한 접근이 불가능하다는 단점이 있다.  
때문에 database에서 읽어온 data를 parsing 하는 과정을 사용하는 측에서 직접 구현해야 해서 번거롭다고 생각하여 방법을 준비했다.  
  
### 1) Recursive template
가장 먼저 든 방법은 recursive를 통해 tuple의 get을 runtime에 모든 index에 대해 생성하는 방법이었다.  
결론부터 말하자면 후술할 방법에 비해 좀 더 generic하게 사용이 가능하여 tuple의 길이가 너무 길지만 않는다면 이 방법을 사용하는 것이 좋을 것 같다.
  
Idea는 매우 단순하다. std::get을 runtime에 helper class를 통해 모두 생성해두고 함수를 통해 접근하는 것이다.
```cpp
template<size_t TupleSize>
struct runtime_get_helper
{
    template<typename Tuple>
    static auto get(Tuple& tuple, size_t idx) -> decltype(auto)
    {
        if (idx == TupleSize - 1)
            return std::get<TupleSize - 1>(tuple);
        else runtime_get_helper<TupleSize - 1>::get(tuple, idx);
    }
};
  
template<>
struct runtime_get_helper<0>
{
    template<typename Tuple>
    static auto get(Tuple& tuple, size_t idx) -> decltype(auto)
    {
        assert(false);
    };
};
  
template<typename... T>
auto runtime_get(const std::tuple<T...>& tuple, size_t idx) -> decltype(auto)
{
    return runtime_get_helper<sizeof...(T)>::get(tuple, idx);
}
  
template<typename... T>
auto runtime_get(std::tuple<T...>& tuple, size_t idx) -> decltype(auto)
{
    return runtime_get_helper<sizeof...(T)>::get(tuple, idx);
}
```
  
### 2) std::index_sequence 사용
위 방법으로도 구현에 문제 없이 사용이 가능하다.  
하지만 새로운 방법은 없는지 조사를 해봤고 std::index_sequence를 사용하여 구현하는 방법이 있었다.  
- tuple을 다루는데 있어서 많이 사용한다고 한다.  
  
Reference > <https://accu.org/journals/overload/25/139/williams_2382/>  
  
#### (1) Function type 결정
함수는 기본적으로 하나의 type만 반환이 가능하기 때문에 tuple이 모두 동일한 type을 가져야 한다고 한다.  
(다행히 MySQL을 통해 받아오는 data가 모두 string을 기반으로 하고 있기 때문에 이 부분은 문제가 되지 않는다.)
  
Type이 결정되어 있기 때문에 tuple의 아무 element의 type을 사용해도 되지만 여기에선 첫번째 element의 type을 사용하기로 한다.  
  
std::tuple_element를 이용해서 tuple의 element를 가져올 수 있는데 위에서 정의한 limitation에 따라 0번째 index의 element를 가져와 type을 반환하도록 한다.  
<https://en.cppreference.com/w/cpp/utility/tuple/tuple_element>  
  
```cpp
template<typename Tuple>
constexpr auto runtime_get(Tuple&& t, size_t index) -> typename std::tuple_element<0, typename std::remove_reference<Tuple>::type>::type&;
```
  
#### (2) n번째 element 반환
Recursive 방법과 달리 std::index_sequence를 통해 runtime에 size에 맞는 table을 생성하고 해당 table에 std::get을 할당하는 방법이다.
  
```cpp
// tuple의 variadic parameter의 길이를 받도록 구성
template<typename Tuple, typename Indices = std::make_index_sequence<std::tuple_size<Tuple>::value>>
struct runtime_get_func_table;
  
template<typename Tuple, size_t ... Indices>
struct runtime_get_func_table<Tuple, std::index_sequence<Indices...>>
{
    using return_type = typename std::tuple_element<0, Tuple>::type&;
    using get_func_ptr = return_type(*)(Tuple&) noexcept;
  
    // table[i]에 std::get<i>를 할당
    static constexpr get_func_ptr table[std::tuple_size<Tuple>::value] =
    {
        &std::get<Indices>...
    };
};
  
// 위에서 구성한 helper를 통해 실제 table을 구성
template<typename Tuple, size_t ... Indices>
constexpr typename
runtime_get_func_table<Tuple, std::index_sequence<Indices...>>::get_func_ptr
runtime_get_func_table<Tuple, std::index_sequence<Indices...>>::table[std::tuple_size<Tuple>::value];
```
  
#### (3) 이점은 ?
Recursive를 통해 구성하게 되면 당연히 variadic parameter의 길이에 따라 depth가 깊어질 수 있으나, table을 생성함으로써 그런 문제를 해결할 수 있다.
하지만 tuple이 여러 type으로 구성된다면 runtime_get을 더 변경해야해서 code가 더 복잡해 질 것이다.
  
현재는 MySQL을 통해 받아오는 data의 field가 많지 않기 때문에 Recursive를 이용해서 구현하는 것으로 결정했다.
  
***
  