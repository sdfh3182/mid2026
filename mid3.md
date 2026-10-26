<img width="1123" height="349" alt="image" src="https://github.com/user-attachments/assets/da114ce2-378e-4d09-9205-bb1656636378" />

3번 답


#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main(void)
{
    int answer[3];
    int guess[3];
    int strike, ball;
    int i, j;
    int count = 0;
    int duplicate;

    srand((unsigned int)time(NULL));

    // 중복되지 않는 세 자리 정답 생성
    for (i = 0; i < 3; i++)
    {
        do
        {
            duplicate = 0;
            answer[i] = rand() % 9 + 1;

            for (j = 0; j < i; j++)
            {
                if (answer[i] == answer[j])
                {
                    duplicate = 1;
                    break;
                }
            }
        } while (duplicate);
    }

    printf("=== 숫자 야구 게임 ===\n");
    printf("서로 다른 1~9 사이의 숫자 3개를 맞혀 보세요!\n");

    while (1)
    {
        printf("\n세 자리 숫자를 입력하세요: ");
        scanf("%1d%1d%1d",
            &guess[0], &guess[1], &guess[2]);

        // 입력 숫자 중복 확인
        if (guess[0] == guess[1] ||
            guess[0] == guess[2] ||
            guess[1] == guess[2] ||
            guess[0] < 1 || guess[0] > 9 ||
            guess[1] < 1 || guess[1] > 9 ||
            guess[2] < 1 || guess[2] > 9)
        {
            printf("서로 다른 1~9 사이의 숫자를 입력하세요.\n");
            continue;
        }

        count++;
        strike = 0;
        ball = 0;

        // 정답과 입력 숫자 비교
        for (i = 0; i < 3; i++)
        {
            for (j = 0; j < 3; j++)
            {
                if (answer[i] == guess[j])
                {
                    if (i == j)
                        strike++;
                    else
                        ball++;
                }
            }
        }

        printf("%d스트라이크 %d볼\n", strike, ball);

        if (strike == 3)
        {
            printf("정답입니다! 총 %d번 만에 맞혔습니다.\n", count);
            break;
        }

        if (strike == 0 && ball == 0)
            printf("아웃!\n");
    }

    return 0;
}
