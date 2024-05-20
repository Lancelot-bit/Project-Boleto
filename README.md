import sys
from PyQt5.QtWidgets import QApplication, QMainWindow, QVBoxLayout, QWidget, QTableWidget, QTableWidgetItem, QLineEdit, QPushButton, QMessageBox
import mysql.connector
from mysql.connector import Error
from datetime import datetime

class MainWindow(QMainWindow):
    def __init__(self):
        super().__init__()

        # Configurar a janela principal
        self.setWindowTitle("Dados do Banco de Dados")
        self.setGeometry(100, 100, 600, 400)

        # Layout principal
        self.layout = QVBoxLayout()

        # Campos de entrada
        self.nome_input = QLineEdit(self)
        self.nome_input.setPlaceholderText("Nome")
        self.layout.addWidget(self.nome_input)

        self.numboleto_input = QLineEdit(self)
        self.numboleto_input.setPlaceholderText("Número Boleto")
        self.layout.addWidget(self.numboleto_input)

        self.data_validade_input = QLineEdit(self)
        self.data_validade_input.setPlaceholderText("Validade (dd/mm/yyyy)")
        self.layout.addWidget(self.data_validade_input)

        # Botão para adicionar dados
        self.add_button = QPushButton("Adicionar", self)
        self.add_button.clicked.connect(self.add_data)
        self.layout.addWidget(self.add_button)

        # Tabela para exibir os dados
        self.tableWidget = QTableWidget()
        self.layout.addWidget(self.tableWidget)

        # Configurar a tabela
        self.tableWidget.setColumnCount(3)
        self.tableWidget.setHorizontalHeaderLabels(["Nome Completo", "Número Boleto", "Validade"])

        # Conectar ao banco de dados e recuperar os dados
        self.retrieve_data()

        # Definir o layout principal
        central_widget = QWidget()
        central_widget.setLayout(self.layout)
        self.setCentralWidget(central_widget)

    def retrieve_data(self):
        try:
            # Conectar ao banco de dados
            conexao = mysql.connector.connect(
                host="localhost",
                user="root",  # Usuário padrão do XAMPP
                password="",  # Senha padrão do XAMPP
                database="meubanco"
            )

            if conexao.is_connected():
                # Cursor para executar consultas SQL
                cursor = conexao.cursor()
                cursor.execute("SELECT nomecompleto, numboleto, data_validade FROM usuarios")

                # Recuperar os resultados
                resultados = cursor.fetchall()

                # Preencher a tabela com os dados
                self.tableWidget.setRowCount(len(resultados))
                for row, (nomecompleto, numboleto, data_validade) in enumerate(resultados):
                    self.tableWidget.setItem(row, 0, QTableWidgetItem(nomecompleto))
                    self.tableWidget.setItem(row, 1, QTableWidgetItem(numboleto))
                    # Verificar se data_validade é None antes de formatar
                    if data_validade:
                        data_validade_str = data_validade.strftime("%d/%m/%Y")
                    else:
                        data_validade_str = ""
                    self.tableWidget.setItem(row, 2, QTableWidgetItem(data_validade_str))

        except Error as e:
            print(f"Erro ao conectar ao banco de dados: {e}")

        finally:
            # Fechar cursor e conexão
            if conexao.is_connected():
                cursor.close()
                conexao.close()

    def add_data(self):
        nomecompleto = self.nome_input.text()
        numboleto = self.numboleto_input.text()
        data_validade = self.data_validade_input.text()

        if not nomecompleto or not numboleto or not data_validade:
            QMessageBox.warning(self, "Entrada Inválida", "Por favor, preencha todos os campos: Nome, Número do Boleto e Validade")
            return

        try:
            # Converter a data de string para o formato de data do MySQL
            data_validade_convertida = datetime.strptime(data_validade, "%d/%m/%Y").strftime("%Y-%m-%d")

            # Conectar ao banco de dados
            conexao = mysql.connector.connect(
                host="localhost",
                user="root",  # Usuário padrão do XAMPP
                password="",  # Senha padrão do XAMPP
                database="meubanco"
            )

            if conexao.is_connected():
                cursor = conexao.cursor()
                cursor.execute("INSERT INTO usuarios (nomecompleto, numboleto, data_validade) VALUES (%s, %s, %s)", (nomecompleto, numboleto, data_validade_convertida))
                conexao.commit()

                QMessageBox.information(self, "Sucesso", "Dados adicionados com sucesso")

                # Limpar campos de entrada
                self.nome_input.clear()
                self.numboleto_input.clear()
                self.data_validade_input.clear()

                # Atualizar tabela
                self.retrieve_data()

        except Error as e:
            QMessageBox.critical(self, "Erro", f"Erro ao conectar ao banco de dados: {e}")

        finally:
            if conexao.is_connected():
                cursor.close()
                conexao.close()

if __name__ == "__main__":
    app = QApplication(sys.argv)
    mainWindow = MainWindow()
    mainWindow.show()
    sys.exit(app.exec_())
