<img width="1118" height="405" alt="image" src="https://github.com/user-attachments/assets/f48d6b41-babc-4abe-9e93-628e044031b0" />

2번 문제 답

#include <stdio.h>
#include "sqlite3.h"

int main(void)
{
    sqlite3* db;
    char* errMsg = NULL;
    int result;

    // 1. 데이터베이스 열기 또는 생성
    result = sqlite3_open("library.db", &db);

    if (result != SQLITE_OK)
    {
        printf("데이터베이스 열기 실패: %s\n",
            sqlite3_errmsg(db));
        sqlite3_close(db);
        return 1;
    }

    printf("데이터베이스 연결 성공!\n");

    // 2. 테이블 생성 및 데이터 입력
    const char* sql =
        "CREATE TABLE IF NOT EXISTS Borrow ("
        "id INTEGER PRIMARY KEY AUTOINCREMENT,"
        "student TEXT NOT NULL,"
        "book TEXT NOT NULL,"
        "status TEXT NOT NULL);"

        "INSERT INTO Borrow (student, book, status) "
        "SELECT '김민수', 'C언어', '대여중' "
        "WHERE NOT EXISTS (SELECT 1 FROM Borrow WHERE student='김민수');"

        "INSERT INTO Borrow (student, book, status) "
        "SELECT '이서연', '데이터베이스', '반납완료' "
        "WHERE NOT EXISTS (SELECT 1 FROM Borrow WHERE student='이서연');"

        "INSERT INTO Borrow (student, book, status) "
        "SELECT '박지훈', '파이썬', '대여중' "
        "WHERE NOT EXISTS (SELECT 1 FROM Borrow WHERE student='박지훈');"

        "INSERT INTO Borrow (student, book, status) "
        "SELECT '최유진', '컴퓨터구조', '대여중' "
        "WHERE NOT EXISTS (SELECT 1 FROM Borrow WHERE student='최유진');";

    result = sqlite3_exec(db, sql, NULL, NULL, &errMsg);

    if (result != SQLITE_OK)
    {
        printf("SQL 실행 오류: %s\n", errMsg);
        sqlite3_free(errMsg);
        sqlite3_close(db);
        return 1;
    }

    printf("\n[전체 도서 대여 목록]\n");

    // 3. 전체 데이터 검색
    const char* query =
        "SELECT id, student, book, status FROM Borrow;";

    sqlite3_stmt* stmt;

    result = sqlite3_prepare_v2(
        db, query, -1, &stmt, NULL);

    if (result != SQLITE_OK)
    {
        printf("검색 오류: %s\n", sqlite3_errmsg(db));
        sqlite3_close(db);
        return 1;
    }

    while (sqlite3_step(stmt) == SQLITE_ROW)
    {
        printf("%d | %s | %s | %s\n",
            sqlite3_column_int(stmt, 0),
            (const char*)sqlite3_column_text(stmt, 1),
            (const char*)sqlite3_column_text(stmt, 2),
            (const char*)sqlite3_column_text(stmt, 3));
    }

    sqlite3_finalize(stmt);

    // 4. 학생 이름으로 검색
    printf("\n[김민수 검색 결과]\n");

    query =
        "SELECT student, book, status "
        "FROM Borrow WHERE student = '김민수';";

    result = sqlite3_prepare_v2(
        db, query, -1, &stmt, NULL);

    if (result != SQLITE_OK)
    {
        printf("검색 오류: %s\n", sqlite3_errmsg(db));
        sqlite3_close(db);
        return 1;
    }

    while (sqlite3_step(stmt) == SQLITE_ROW)
    {
        printf("학생: %s\n",
            (const char*)sqlite3_column_text(stmt, 0));
        printf("도서: %s\n",
            (const char*)sqlite3_column_text(stmt, 1));
        printf("상태: %s\n",
            (const char*)sqlite3_column_text(stmt, 2));
    }

    sqlite3_finalize(stmt);

    // 5. 종료
    sqlite3_close(db);

    printf("\n프로그램 종료!\n");

    return 0;
}
