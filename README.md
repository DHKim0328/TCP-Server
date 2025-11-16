    //TCP 클라이언트
    #define _WINSOCK_DEPRECATED_NO_WARNINGS
    #include <winsock2.h>
    #include <iostream>
    #include <fstream>
    #include <string>

    #pragma comment(lib, "ws2_32.lib")

    int main() {
        WSADATA wsaData;
        WSAStartup(MAKEWORD(2, 2), &wsaData);

        SOCKET clientSock = socket(AF_INET, SOCK_STREAM, 0);

        sockaddr_in serverAddr;
        serverAddr.sin_family = AF_INET;
        serverAddr.sin_addr.s_addr = inet_addr("127.0.0.1");
        serverAddr.sin_port = htons(9000);

        if (connect(clientSock, (SOCKADDR*)&serverAddr, sizeof(serverAddr)) == SOCKET_ERROR) {
            std::cerr << "[클라이언트] 서버 연결 실패\n";
            closesocket(clientSock);
            WSACleanup();
            return 1;
        }

        std::cout << "[클라이언트] 서버 연결됨\n";

        char buffer[1024];
        std::string input;

        while (true) {
            std::cout << "\nlist를 입력하여 파일 list 수신: ";
            std::cin >> input;

            // list 요청
            if (strncmp(input.c_str(), "list", 4) == 0) {

                send(clientSock, "list", 4, 0);

                // 목록 수신
                int recvLen = recv(clientSock, buffer, sizeof(buffer), 0);
                if (recvLen <= 0) {
                    std::cout << "[클라이언트] 목록 수신 실패\n";
                    continue;
                }

                buffer[recvLen] = '\0';
                std::cout << "\nlist : " << buffer;
    
                // 파일 요청
                std::cout << "\n파일명 입력: ";
                std::cin >> input;

                send(clientSock, input.c_str(), input.size(), 0);

                // 파일 수신 준비
                std::string outname = input;
                std::ofstream outFile(outname, std::ios::binary);

                while (true) {
                    int bytes = recv(clientSock, buffer, sizeof(buffer), 0);

                    if (bytes <= 0)
                        break;

                    if (strncmp(buffer, "None", 4) == 0) {
                        std::cout << "[클라이언트] 서버: None!\n";
                        outFile.close();
                        remove(outname.c_str());
                        break;
                    }

                    outFile.write(buffer, bytes);
                outFile.close();
                std::cout << "[클라이언트] " << outname << "파일 수신 완료\n";
                }
                break;
            }
            else {
                std::cout << "옳바르지 않은 수식입니다.\n";
            };
        }

    closesocket(clientSock);
    WSACleanup();
    return 0;
}
