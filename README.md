# Fitverso — Arquitetura e Plano de Ataque

**Status:** 2026-10-25 | Repo: maxteel122343/fitverso

## Visão

Mundo virtual compartilhado onde vários usuários fazem exercícios em tempo real. Câmera captura o movimento, esqueleto 3D aparece no avatar, IA personalizada guia os exercícios do dataset e compara a execução com referência — feedback de "fez certo / fez errado" no loop.

Diferencial vs Peloton/Mirror: presença social, competição e accountability em grupo. Apps individuais com tela bonita; Fitverso é um espaço social.

## Ordem de Ataque (nova estratégia)

### Fase 1 — Validar a base de pose (AGORA)
**[Good-GYM](https://github.com/maxteel122343/Good-GYM)** (fork de yo-WASSUP/Good-GYM)
- RTMPose via rtmlib, roda em **CPU** (sem GPU)
- Contagem de reps automática, 11 exercícios (agachamento, flexão, abdominal, rosca, elevação lateral, etc.)
- Visualização de esqueleto + medição de ângulos + correção de postura
- PyQt5, ONNX Runtime, MIT license
- **Dia 1:** instalar, abrir webcam, medir latência frame→keypoint. Meta: < 100 ms.

### Fase 2 — Plano B de pose
**[Pose-Comparison](https://github.com/maxteel122343/Pose-Comparison)** (fork de TodayAnalyze/Pose-Comparison)
- Três pipelines lado a lado: MediaPipe, YOLOv11, OpenPose
- Mesmo código de contagem e correção guiada pra cada um
- Usar se RTMPose não entregar latência ou precisão no Fitverso

### Fase 3 — Camada de comparação com coach (diferencial)
**[MemryX_eXamples](https://github.com/maxteel122343/MemryX_eXamples)** → `fun_projects/fitness_mirror_yolov8/`
- YOLOv8m-Pose + **Dynamic Time Warping (DTW)** comparando movimento ao vivo vs vídeo do coach
- Score de similaridade em tempo real + contagem com histerese
- É a peça que transforma "contar reps" em "avaliar forma" — o cérebro do Fitverso

## Kit Completo (repos anteriores, papéis mantidos)

| Repo | Papel | Quando entra |
|------|-------|--------------|
| trt_pose | Pose 2D tempo real (TensorRT/Jetson) | Se Good-GYM for lento em hardware fraco |
| GEM-X | Pose 3D de qualidade (SOMA) | Renderizar avatar no mundo virtual |
| soma-retargeter | Humano→robô retargeting | Animar personagem 3D copiando o usuário |
| blosm | Google 3D Tiles + OSM + terreno | Mundo virtual com cidade real (Tourverse) |
| exercises-dataset | 1324 exercícios, GIFs, 10 idiomas | Fonte de verdade dos movimentos |
| SuperSplat | Splat 3D em tempo real | Render leve pra PWA |
| openGym | UX de referência | Guiado animado self-hosted |
| 3d-model-anything | Geração/edição 3D | Assets do mundo virtual |

## Pipeline com latência estimada

1. Webcam → frame: ~16 ms (60 fps)
2. RTMPose inferência (CPU): ~30–60 ms
3. Cálculo de ângulos + contagem: ~5 ms
4. Render esqueleto: ~10 ms
5. **Total: ~60–90 ms** — dentro da meta de 100 ms

Acima de 200 ms a sensação de espelho quebra.

## Prompt pro GitHub Copilot (primeiro protótipo)

```
Cria um protótipo Python minimal do Fitverso usando o código do Good-GYM como base.

Requisitos:
1. Abre a webcam e roda RTMPose (via rtmlib) em tempo real
2. Desenha o esqueleto 17 keypoints COCO sobre o frame com OpenCV
3. Implementa contagem de reps de AGACHAMENTO usando os keypoints hip-knee-ankle
   (histerese: conta 1 rep quando o joelho passa de 90° pra cima e volta)
4. Mede e imprime no terminal a latência de cada frame: captura → keypoint → render
5. Mostra um contador de reps na tela
6. Estrutura de pastas limpa: src/pose.py, src/counters.py, src/main.py
7. requirements.txt com rtmlib, opencv-python, numpy
8. README com instruções de instalação e como rodar

Não adiciona GUI PyQt5 — só OpenCV. Foco em latência medida e código legível.
```

## Próximos Passos

1. Rodar Good-GYM, medir latência real
2. Se < 100 ms: plugar dataset de exercícios
3. Se > 100 ms: testar MediaPipe do Pose-Comparison
4. Estudar o DTW do MemryX pra camada de score
5. Integrar com Tourverse quando a base de pose estiver estável

## Decisões em Aberto

- IA personalizada roda local ou remoto?
- Avatar 3D: GEM-X 3D ou retargeting SOMA?
- Mundo virtual: Blender render ou Three.js no browser?
- Monetização: créditos, assinatura, marketplace de e-books?
