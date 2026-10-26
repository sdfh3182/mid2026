
#include <stdio.h>

int main(void)
{
    int N;

    printf("높이 N을 입력하세요: ");
    scanf("%d", &N);

    for (int i = 1; i <= N; i++)
    {
        // 왼쪽 공백 출력
        for (int j = 1; j <= N - i; j++)
        {
            printf(" ");
        }

        // 홀수 행은 *, 짝수 행은 #
        for (int j = 1; j <= 2 * i - 1; j++)
        {
            if (i % 2 == 1)
                printf("*");
            else
                printf("#");
        }

        printf("\n");
    }

    return 0;
}
