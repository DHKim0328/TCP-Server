#define _WINSOCK_DEPRECATED_NO_WARNINGS
#include <winsock2.h>
#include <iostream>
#include <fstream>
#include <string>

#pragma comment(lib, "ws2_32.lib")

int main() {
    WSADATA wsaData;
    WSAStartup(MAKEWORD(2, 2), &wsaData);

    SOCKET serverSock = socket(AF_INET, SOCK_STREAM, 0);

    sockaddr_in serverAddr;
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_addr.s_addr = INADDR_ANY;
    serverAddr.sin_port = htons(9000);

    bind(serverSock, (SOCKADDR*)&serverAddr, sizeof(serverAddr));
    listen(serverSock, 1);

    std::cout << "[서버] 클라이언트 접속 대기 중...\n";

    sockaddr_in clientAddr;
    int clientAddrSize = sizeof(clientAddr);
    SOCKET clientSock = accept(serverSock, (SOCKADDR*)&clientAddr, &clientAddrSize);

    std::cout << "[서버] 클라이언트 접속!\n";

    char buffer[1024];

    while (true) {
        // 클라이언트 요청 수신
        int recvlen = recv(clientSock, buffer, sizeof(buffer), 0);
        if (recvlen <= 0) break;

        buffer[recvlen] = '\0';
        std::string cmd = buffer;

        // list 명령 처리 (파일 목록 전송)
        if (strncmp(buffer, "list", 4) == 0) {
            std::cout << "[서버] 클라이언트 list 요청"<< "\n";
            const char* fileList = "a.txt, b.txt, c.png, d.png\n";
            send(clientSock, fileList, strlen(fileList), 0);
            continue;
        }

        // 파일 요청 처리
        std::ifstream file(cmd, std::ios::binary);

        if (!file) {
            const char* errorMsg = "None";
            send(clientSock, errorMsg, strlen(errorMsg), 0);
            std::cout << "[서버] None! " << "\n";
            break;
        }

        else std::cout << "[서버] " << cmd << "파일 전송 !" << "\n";

        while (!file.eof()) {
            file.read(buffer, sizeof(buffer));
            int bytes = file.gcount();
            if (bytes > 0) {
                send(clientSock, buffer, bytes, 0);
            }
        }

        file.close();
        break;
    }

    closesocket(clientSock);
    closesocket(serverSock);
    WSACleanup();

    return 0;
}
