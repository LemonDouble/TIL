# 가변 인자 함수 처리

배운 내용 (한줄 요약): va_start, va_arg, va_end
분류: C
자료 링크: https://dojang.io/mod/page/view.php?id=578
작성일시: 2021년 6월 28일 오후 4:14

## 가변 인자 함수 만들기

- GCC에서는 가변 인자로 받은 값이 int보다 작다면 int로 float라면 double로 지정한 후 type casting 해 줘야 함.

```c
#include <stdio.h>
#include <stdarg.h>    // va_list, va_start, va_arg, va_end가 정의된 헤더 파일

void printValues(char *types, ...)    // 가변 인자의 자료형을 받음, ...로 가변 인자 설정
{
    va_list ap;    // 가변 인자 목록
    int i = 0;

    va_start(ap, types);        // types 문자열에서 문자 개수를 구해서 가변 인자 포인터 설정
    while (types[i] != '\0')    // 가변 인자 자료형이 없을 때까지 반복
    {
        switch (types[i])       // 가변 인자 자료형으로 분기
        {
        case 'i':                                // int형일 때
            printf("%d ", va_arg(ap, int));      // int 크기만큼 값을 가져옴
                                                 // ap를 int 크기만큼 순방향으로 이동
            break;
        case 'd':                                // double형일 때
            printf("%f ", va_arg(ap, double));   // double 크기만큼 값을 가져옴
                                                 // ap를 double 크기만큼 순방향으로 이동
            break;
        case 'c':                                // char형 문자일 때
            printf("%c ", (char)va_arg(ap, int));     // char 크기만큼 값을 가져옴
                                                 // ap를 char 크기만큼 순방향으로 이동
            break;
        case 's':                                // char *형 문자열일 때
            printf("%s ", (char*)va_arg(ap, int *));   // char * 크기만큼 값을 가져옴
                                                 // ap를 char * 크기만큼 순방향으로 이동
            break;
        default:
            break;
        }
        i++;
    }
    va_end(ap);    // 가변 인자 포인터를 NULL로 초기화

    printf("\n");    // 줄바꿈
}

int main()
{
    printValues("i", 10);                                       // 정수
    printValues("ci", 'a', 10);                                 // 문자, 정수
    printValues("dci", 1.234567, 'a', 10);                      // 실수, 문자, 정수
    printValues("sicd", "Hello, world!", 10, 'a', 1.234567);    // 문자열, 정수, 문자, 실수

    return 0;
}
```

- va_list : 가변 인자 목록, 가변 인자의 메모리 주소를 저장하는 포인터
- va_start : 가변 인자를 가져올 수 있도록 포인터를 설정
- va_arg : 가변 인자 포인터에서 특정 자료형 크기만큼 값을 가져옴
- va_end : 가변 인자 처리가 끝났을 때 포인터를 NULL로 초기화

![https://dojang.io/pluginfile.php/642/mod_page/content/27/unit66-4.png](https://dojang.io/pluginfile.php/642/mod_page/content/27/unit66-4.png)