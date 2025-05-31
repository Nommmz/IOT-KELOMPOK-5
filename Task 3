#include "olcWebSocketServer.h"
#include <iostream>
#include <fstream>
#include <string>
#include <sstream>
#include <iomanip>
#include <ctime>

using namespace std;

class Server : public olc::WebSocketServer
{
public:
    Server()
        : olc::WebSocketServer(12345, "fullchain.pem", "privkey.pem", 0.1f)
    {
        logFile.open("data_log.txt", ios::app); // Buka file log untuk menulis data historis
        if (!logFile.is_open())
            cerr << "[ERROR] Tidak bisa membuka file log!\n";
    }

    ~Server()
    {
        if (logFile.is_open())
            logFile.close();
    }

    // Saat client tersambung
    bool OnClientConnect(const ws_connection& client) override
    {
        cout << "[" << client.address->first << ":" << client.uid << "] Connected\n";
        return true;
    }

    // Saat client terputus
    void OnClientDisconnect(const ws_connection& client) override
    {
        cout << "[" << client.address->first << ":" << client.uid << "] Disconnected\n";
    }

    // Saat data masuk dari client (ESP32, dll)
    bool OnClientIncoming(const ws_connection& client, const string& data, bool text_mode) override
    {
        if (text_mode && data.size() > 5 && data.substr(0, 5) == "drb42")
        {
            string valueStr = data.substr(5);

            if (valueStr == "15") // Anggap 15 sebagai sinyal idle, tidak dicatat
                return true;

            try
            {
                float y = stof(valueStr); // Konversi string ke float
                string json = "{\"x\":" + to_string(nTick++) + ",\"y\":" + to_string(y) + "}";

                WriteToAll(json);   // Kirim data JSON ke semua client WebSocket
                cout << json << endl;
                LogData(y);         // Simpan ke file log
            }
            catch (const exception& e)
            {
                cerr << "[ERROR] Gagal parsing: " << e.what() << " | Data: " << valueStr << endl;
            }
        }
        return true;
    }

    // Validasi data awal koneksi client
    bool OnValidateIncoming(const string& data) override
    {
        return data.size() > 5 && data.substr(0, 5) == "drb42";
    }

    // Dipanggil setiap frame (tidak digunakan saat ini)
    bool OnUpdate(const float fElapsedTime) override
    {
        return true;
    }

private:
    int nTick = 0;
    ofstream logFile;

    // Simpan data ke file log dengan timestamp
    void LogData(float y)
    {
        if (!logFile.is_open()) return;

        auto t = time(nullptr);
        auto tm = *localtime(&t);

        ostringstream oss;
        oss << put_time(&tm, "%Y-%m-%d %H:%M:%S");

        logFile << oss.str() << ", " << y << " A" << endl;
    }
};

int main()
{
    Server server;
    server.Start();
    return 0;
}
