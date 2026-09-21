IoT_PI: Monitoramento Ambiental com ThingSpeak 🌡️☁️

📋 Sobre o Projeto

Este projeto consiste em um sistema de Internet das Coisas (IoT) desenvolvido para monitorar condições climáticas locais. Utilizando um microcontrolador com suporte a WiFi (como o ESP32 ou ESP8266) e um sensor DHT11, o sistema coleta dados de temperatura e umidade e os transmite em tempo real para a nuvem utilizando o protocolo MQTT. Os dados são enviados para a plataforma ThingSpeak, permitindo a visualização e análise contínua das informações em painéis (dashboards).

⚙️ Configuração e Credenciais

Para proteger dados sensíveis, o projeto exige um arquivo de configuração separado. Para executar o código localmente, você deve criar um arquivo chamado arduino_secrets.h no mesmo diretório do arquivo principal (.ino ou .cpp) e preenchê-lo com as suas credenciais de rede e do ThingSpeak:

#define SECRET_SSID "NOME_DA_SUA_REDE_WIFI"
#define SECRET_PASS "SENHA_DO_SEU_WIFI"
#define SECRET_MQTT_CLIENT_ID "SEU_MQTT_CLIENT_ID"
#define SECRET_MQTT_USERNAME "SEU_MQTT_USERNAME"
#define SECRET_MQTT_PASSWORD "SUA_MQTT_PASSWORD"
#define SECRET_CHANNEL_ID 1234567


⚠️ Importante: Lembre-se de adicionar o arquivo arduino_secrets.h ao seu .gitignore para não expor suas senhas e chaves de API ao enviar o código para o GitHub.

🔍 Avaliação Técnica do Código

✨ Boas Práticas e Pontos Fortes

Segurança de Credenciais: Excelente decisão arquitetural ao isolar variáveis sensíveis no arquivo arduino_secrets.h através de diretivas de pré-processador (#define). Isso mantém o repositório seguro e o código principal limpo.

Sistema Resiliente: O código possui validações contínuas de conexão dentro do loop() (if (WiFi.status() != WL_CONNECTED) e if (!client.connected())). Se a rede ou o servidor caírem, o dispositivo tenta se reconectar automaticamente em vez de travar.

Tratamento de Falhas do Sensor: A checagem if (isnan(temperatura) || isnan(umidade) || (temperatura == 0 && umidade == 0)) é muito bem aplicada. O DHT11 costuma falhar ocasionalmente; essa lógica impede que leituras corrompidas ou zeradas sejam enviadas para o banco de dados.

Conformidade com a API: O uso do intervalo de 16 segundos no final do ciclo demonstra conhecimento sobre os limites técnicos da API gratuita do ThingSpeak, que exige um intervalo mínimo de 15 segundos entre as requisições.

🛠️ Pontos de Melhoria e Refatoração

Substituição do delay() Bloqueante: A função delay(16000) paralisa completamente o microcontrolador por 16 segundos. Isso impede que a biblioteca MQTT (client.loop()) processe mensagens ou mantenha o ping com o servidor em segundo plano, o que pode causar desconexões fantasmas. Sugestão: Substituir por um temporizador não-bloqueante utilizando a função millis().

Gerenciamento de Memória (Uso de String): A criação dinâmica do tópico com String topico = "channels/" + String(channelID) + "/publish"; pode causar fragmentação na memória heap do microcontrolador após dias rodando ininterruptamente. Sugestão: Como você já utilizou o sprintf de forma eficiente para criar o payload, aplique a mesma técnica para montar o topico em um array de caracteres fixo (char), eliminando o uso da classe String.
