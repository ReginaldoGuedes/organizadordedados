# organizadordedados
Organize os dados CSV de Lista para Colunas e Colunas Para Lista Facilita muito para usuários de CorelDraw para usar Impressão Mesclada
Link de Download
https://drive.google.com/file/d/1KxQVMpU3cdU0XB2ys5aaQLeANDeGUcZB/view?usp=sharing

    import sys
    import pandas as pd
    from PyQt5.QtWidgets import (QApplication, QMainWindow, QLabel, QLineEdit,
                             QPushButton, QFileDialog, QVBoxLayout, QWidget,
                             QMessageBox, QHBoxLayout, QProgressBar, QComboBox)
    from PyQt5.QtCore import Qt, QPropertyAnimation, QEasingCurve, QEvent, pyqtSignal, QTimer
    from PyQt5.QtGui import QIcon, QPixmap, QScreen
    import os

    def resource_path(relative_path):
    """ Get absolute path to resource, works for dev and for PyInstaller """
    try:
        # PyInstaller creates a temp folder and stores path in _MEIPASS
        base_path = sys._MEIPASS
    except AttributeError:
        base_path = os.path.abspath(".")
    return os.path.join(base_path, relative_path)

    class MainWindow(QMainWindow):
        closedSignal = pyqtSignal()

    def __init__(self):
        super().__init__()
        self.setWindowTitle("Organizador de Dados")
        self.setFixedSize(500, 300)
        self.setWindowIcon(QIcon(resource_path('icon/csv.png')))
        self.setWindowOpacity(1.0)
        self._is_closing = False

        # Definindo o estilo para cantos arredondados
        self.setStyleSheet("""
            QMainWindow {
                border-radius: 10px; /* Ajuste o valor para controlar o raio dos cantos */
                background-color: white; /* Opcional: defina uma cor de fundo para a janela */
            }
        """)

        self.central_widget = QWidget()
        self.setCentralWidget(self.central_widget)
        self.layout = QVBoxLayout()

        # Seletor de Modo
        self.modo_selector = QComboBox()
        self.modo_selector.addItem("Organizar Lista em Colunas")
        self.modo_selector.addItem("Converter Colunas para Lista")
        self.layout.addWidget(self.modo_selector)

        # Seleção de Arquivo
        self.label_arquivo = QLabel("Arquivo CSV:")
        self.entry_arquivo = QLineEdit(placeholderText="Selecione o arquivo CSV")

        # Botões Lado a Lado
        self.botoes_layout = QHBoxLayout()
        self.botao_selecionar = QPushButton("Selecionar Arquivo")
        self.botao_selecionar.clicked.connect(self.selecionar_arquivo)
        self.botao_selecionar.setStyleSheet("background-color: #4CAF50; color: white; border-radius: 5px; padding: 8px;")
        self.botao_transformar = QPushButton("Processar")
        self.botao_transformar.clicked.connect(self.processar_arquivo)
        self.botao_transformar.setStyleSheet("background-color: #008CBA; color: white; border-radius: 5px; padding: 8px;")
        self.botoes_layout.addWidget(self.botao_selecionar)
        self.botoes_layout.addWidget(self.botao_transformar)

        # Barra de Progresso
        self.barra_progresso = QProgressBar()
        self.barra_progresso.setRange(0, 100)
        self.barra_progresso.setValue(0)
        self.barra_progresso.setVisible(False)

        # Informação do Desenvolvedor
        self.dev_label = QLabel("Desenvolvido por Reginaldo Guedes | Gráfica Card | Donate: PIX 11995274171")
        self.dev_label.setAlignment(Qt.AlignCenter)
        self.dev_label.setStyleSheet("font-size: 10px; color: gray;")

        # Layout Principal
        self.layout.addWidget(self.label_arquivo)
        self.layout.addWidget(self.entry_arquivo)
        self.layout.addLayout(self.botoes_layout)
        self.layout.addWidget(self.dev_label)
        self.layout.addWidget(self.barra_progresso)

        self.central_widget.setLayout(self.layout)

        self.fade_out_animation = QPropertyAnimation(self, b"windowOpacity")
        self.fade_out_animation.setDuration(500)
        self.fade_out_animation.setStartValue(1.0)
        self.fade_out_animation.setEndValue(0.0)
        self.fade_out_animation.finished.connect(self._close_app)

    def selecionar_arquivo(self):
        caminho_arquivo, _ = QFileDialog.getOpenFileName(
            self,
            "Selecionar arquivo CSV",
            "",
            "Arquivos CSV (*.csv)"
        )
        if caminho_arquivo:
            self.entry_arquivo.setText(caminho_arquivo)

    def processar_arquivo(self):
        modo = self.modo_selector.currentText()
        caminho_arquivo_entrada = self.entry_arquivo.text()

        if not caminho_arquivo_entrada:
            QMessageBox.showerror(self, "Erro", "Por favor, selecione um arquivo CSV.")
            return

        if modo == "Organizar Lista em Colunas":
            self.organizar_lista_em_colunas(caminho_arquivo_entrada)
        elif modo == "Converter Colunas para Lista":
            self.converter_colunas_para_lista(caminho_arquivo_entrada)

    def organizar_lista_em_colunas(self, caminho_arquivo_entrada):
        caminho_arquivo_saida, _ = QFileDialog.getSaveFileName(
            self,
            "Salvar arquivo como",
            "resultado_colunas.csv",
            "Arquivos CSV (*.csv)"
        )

        if not caminho_arquivo_saida:
            return

        try:
            self.barra_progresso.setVisible(True)
            self.barra_progresso.setValue(10)

            df = pd.read_csv(caminho_arquivo_entrada, encoding='ISO-8859-1')
            self.barra_progresso.setValue(50)

            nomes = df.iloc[:, 0].dropna().tolist()
            self.barra_progresso.setValue(70)

            df_resultado = pd.DataFrame([nomes], columns=[f"Nome{i+1}" for i in range(len(nomes))])
            self.barra_progresso.setValue(90)

            df_resultado.to_csv(caminho_arquivo_saida, index=False)
            self.barra_progresso.setValue(100)

            QMessageBox.information(self, "Sucesso", f"Lista organizada em colunas e salva em '{caminho_arquivo_saida}'")
            self.barra_progresso.setVisible(False)
            self.barra_progresso.setValue(0)

        except FileNotFoundError:
            QMessageBox.showerror(self, "Erro", "Arquivo de entrada não encontrado.")
            self.barra_progresso.setVisible(False)
            self.barra_progresso.setValue(0)
        except Exception as e:
            QMessageBox.showerror(self, "Erro", f"Ocorreu um erro: {e}")
            self.barra_progresso.setVisible(False)
            self.barra_progresso.setValue(0)

    def converter_colunas_para_lista(self, caminho_arquivo_entrada):
        caminho_arquivo_saida, _ = QFileDialog.getSaveFileName(
            self,
            "Salvar lista como",
            "resultado_lista.csv",
            "Arquivos CSV (*.csv)"
        )

        if not caminho_arquivo_saida:
            return

        try:
            self.barra_progresso.setVisible(True)
            self.barra_progresso.setValue(10)

            df = pd.read_csv(caminho_arquivo_entrada, encoding='ISO-8859-1')
            self.barra_progresso.setValue(50)

            # Converte todas as colunas para uma única lista
            lista_completa = []
            for col in df.columns:
                lista_completa.extend(df[col].dropna().tolist())

            df_resultado = pd.DataFrame({'Nomes': lista_completa})
            self.barra_progresso.setValue(90)

            df_resultado.to_csv(caminho_arquivo_saida, index=False)
            self.barra_progresso.setValue(100)

            QMessageBox.information(self, "Sucesso", f"Colunas convertidas em lista e salvas em '{caminho_arquivo_saida}'")
            self.barra_progresso.setVisible(False)
            self.barra_progresso.setValue(0)

        except FileNotFoundError:
            QMessageBox.showerror(self, "Erro", "Arquivo de entrada não encontrado.")
            self.barra_progresso.setVisible(False)
            self.barra_progresso.setValue(0)
        except Exception as e:
            QMessageBox.showerror(self, "Erro", f"Ocorreu um erro: {e}")
            self.barra_progresso.setVisible(False)
            self.barra_progresso.setValue(0)

    def closeEvent(self, event):
        if not self._is_closing:
            self._is_closing = True
            self.fade_out_animation.start()
            event.ignore()
        else:
            event.accept()

    def _close_app(self):
        self.closedSignal.emit()

    class SplashScreen(QWidget):
      def __init__(self, main_window):
        super().__init__()
        self.main_window = main_window
        self.setWindowTitle("Carregando...")
        self.setStyleSheet("background-color: #f0f0f0;")
        self.setWindowOpacity(0.0)
        self.setWindowFlag(Qt.FramelessWindowHint)

        # Carregar e redimensionar a imagem
        logo_path = resource_path('icon/csv.png')
        self.logo_pixmap = QPixmap(logo_path).scaledToHeight(100, Qt.SmoothTransformation)
        self.logo_label = QLabel(self)
        self.logo_label.setPixmap(self.logo_pixmap)
        self.logo_label.setAlignment(Qt.AlignCenter)

        # Barra de progresso para o loading
        self.progress_bar = QProgressBar(self)
        self.progress_bar.setRange(0, 100)
        self.progress_bar.setValue(0)

        layout = QVBoxLayout(self)
        layout.addWidget(self.logo_label)
        layout.addWidget(self.progress_bar)
        self.setLayout(layout)

        self.animation = QPropertyAnimation(self, b"windowOpacity")
        self.animation.setDuration(500)
        self.animation.setStartValue(0.0)
        self.animation.setEndValue(1.0)
        self.animation.setEasingCurve(QEasingCurve.OutCubic)
        self.animation.finished.connect(self.start_loading)

        self.load_timer = QTimer(self)
        self.load_timer.timeout.connect(self.update_progress)
        self.load_counter = 0
        self.load_duration = 2000
        self.load_interval = 100

        # Centralizar após a inicialização da interface
        self.showLaterTimer = QTimer()
        self.showLaterTimer.singleShot(0, self.center_screen)

    def center_screen(self):
        screen = QApplication.primaryScreen()
        if screen is not None:
            screen_geometry = screen.geometry()
            window_geometry = self.frameGeometry()
            center_point = screen_geometry.center()
            new_x = center_point.x() - window_geometry.width() / 2
            new_y = center_point.y() - window_geometry.height() / 2
            self.move(int(new_x), int(new_y))

    def showEvent(self, event):
        self.animation.start()

    def start_loading(self):
        self.load_timer.start(self.load_interval)

    def update_progress(self):
        self.load_counter += self.load_interval
        progress = int((self.load_counter / self.load_duration) * 100)
        self.progress_bar.setValue(min(progress, 100))
        if self.load_counter >= self.load_duration:
            self.load_timer.stop()
            self.show_main_window()

    def show_main_window(self):
        self.main_window.show()
        self.close()

    if __name__ == '__main__':
      app = QApplication(sys.argv)
      app_icon = QIcon(resource_path('icon/csv.png'))
      app.setWindowIcon(app_icon)

    main_window = MainWindow()
        splash = SplashScreen(main_window)
        main_window.closedSignal.connect(app.quit)
        splash.show()
        sys.exit(app.exec_())
