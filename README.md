# Instalar dependências necessárias (pode rodar no Colab ou no seu terminal)
!pip install opencv-python-headless moviepy numpy

# Importar bibliotecas
import cv2
import numpy as np
from moviepy.editor import VideoFileClip

# Função para detectar movimento e cortar o vídeo
def process_video(input_video, output_folder):
    # Abrir o vídeo
    cap = cv2.VideoCapture(input_video)

    # Inicializando variáveis
    prev_frame = None
    start_frame = None
    end_frame = None
    clips = []

    # Loop para analisar o vídeo
    frame_count = 0
    while True:
        ret, frame = cap.read()
        if not ret:
            break

        frame_count += 1
        gray_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        gray_frame = cv2.GaussianBlur(gray_frame, (21, 21), 0)

        # Detectando a diferença entre quadros
        if prev_frame is None:
            prev_frame = gray_frame
            continue

        frame_diff = cv2.absdiff(prev_frame, gray_frame)
        _, thresh = cv2.threshold(frame_diff, 25, 255, cv2.THRESH_BINARY)
        contours, _ = cv2.findContours(thresh, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

        # Se movimento for detectado, marca os tempos de início e fim
        if contours:
            if start_frame is None:
                start_frame = frame_count
            end_frame = frame_count

        # Salvar trecho se movimento é detectado
        if start_frame and end_frame and (frame_count - start_frame > 30):  # Minimo de frames de movimento
            clip = VideoFileClip(input_video).subclip(start_frame / 30, end_frame / 30)  # Ajuste conforme FPS
            clips.append(clip)
            start_frame = None
            end_frame = None

        prev_frame = gray_frame

    cap.release()

    # Unir clipes detectados
    final_clip = concatenate_videoclips(clips)
    final_clip.write_videofile(f"{output_folder}/highlight_reel.mp4", codec="libx264")

# Usar a função com o vídeo desejado
input_video = "seu_video.mp4"  # Substitua pelo caminho do seu vídeo
output_folder = "/content/drive/MyDrive"  # Saída no Google Drive ou no seu computador
process_video(input_video, output_folder)

print("Processamento concluído! Highlights exportados.")
