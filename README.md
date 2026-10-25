# Fitverso — Arquitetura do Pipeline

> Conceito: loop fechado de fitness em tempo real
> Captura → Comparação → Correção → Repetição
> Data: 2026-10-25

---

## 1. Visão Geral

O usuário faz movimentos fitness na frente da câmera do celular.
O sistema captura em tempo real, renderiza um esqueleto 3D no mundo virtual,
a IA guia os exercícios (via dataset) e dá feedback se o movimento está certo ou errado.

**Três peças centrais:**
1. **Câmera** → captura de vídeo
2. **Esqueleto 3D** → pose estimation em tempo real
3. **Dataset de exercícios** → fonte de verdade (1324 exercícios, 10 idiomas)

---

## 2. Repositórios do Kit

| Repo | Papel no Fitverso | Fork |
|------|------------------|------|
| `trt_pose` (NVIDIA-AI-IOT) | Pose 2D em tempo real, leve, TensorRT | maxteel122343/trt_pose |
| `GEM-X` (NVlabs) | Pose 3D de qualidade para o avatar virtual | maxteel122343/GEM-X |
| `exercises-dataset` (hasaneyldrm) | 1324 exercícios: GIFs, categorias, grupos musculares, instruções em 10 idiomas, MIT | maxteel122343/exercises-dataset |
| `soma-retargeter` (NVIDIA) | Retargeting de esqueleto humano → articulações de robô/avatar | (avaliar) |
| `openGym` (DuarteSantos8) | Referência de UX: tracker self-hosted, guided workouts animados | maxteel122343/openGym |
| `blosm` (vvoovv) | Importar cidades 3D (Google Tiles + OSM) pro mundo virtual | maxteel122343/blosm |
| `SuperSplat` (PlayCanvas) | Renderização de splat 3D em tempo real | maxteel122343/supersplat |
| `3d-model-anything` | Geração/edição de modelos 3D | maxteel122343/3d-model-anything |

---

## 3. Pipeline (Etapas + Latência Estimada)

```
[Câmera do celular]
    ↓ ~16ms (60fps)
[trt_pose] → keypoints 2D
    ↓ ~10-20ms (TensorRT no Jetson/GPU)
[GEM-X] → pose 3D (SOMA)
    ↓ ~30-50ms
[Camada de Integração] ← AQUI É ONDE TUDO PODE MORRER
    ↓ ~5-10ms
[Comparação com exercício de referência (exercises-dataset)]
    ↓ ~10-20ms
[IA personalizada] → feedback + próximo exercício
    ↓ ~50-100ms (LLM/API)
[Render do avatar no mundo virtual (SuperSplat/PlayCanvas)]
    ↓ ~16ms
[Tela do usuário]
```

**Latência total estimada: ~140-230ms**

- Abaixo de 100ms → sensação de espelho perfeita
- 100-200ms → aceitável, ainda parece responsivo
- Acima de 200ms → quebra a imersão

**Gargalo provável #1:** a chamada da IA personalizada (se for LLM remoto).
**Gargalo provável #2:** GEM-X em dispositivos móveis sem GPU dedicada.

---

## 4. A Camada de Integração (o trabalho real)

Cada repo tem:
- Linguagem diferente (Python, C++, TypeScript)
- Formato de dados diferente (JSON, BVH, CSV, tensors)
- Dependências diferentes

A ponte precisa:
1. Normalizar os keypoints do trt_pose pro formato SOMA do GEM-X
2. Mapear os 1324 exercícios do dataset pra representação de pose de referência
3. Calcular distância/similaridade entre pose do usuário e pose de referência
4. Gerar feedback acionável (não só "errado", mas "joelhos mais pra frente")
5. Decidir o próximo exercício com base no desempenho

**Sugestão:** essa camada em TypeScript/React (alinha com o stack do Tourverse),
rodando como serviço separado que os outros componentes chamam via API local.

---

## 5. Mundo Virtual

Duas abordagens:
- **SuperSplat/PlayCanvas:** splat 3D em tempo real, leve, bom pra PWA
- **blosm + Blender:** cidades 3D georreferenciadas (mais pesado, melhor qualidade)

Pro MVP do Fitverso, SuperSplat parece o caminho: o foco é o esqueleto e o feedback,
não a cidade. A cidade vira diferencial na v2.

---

## 6. Próximos Passos (ordem sugerida)

1. [ ] Fazer trt_pose rodar com webcam local (teste de latência real)
2. [ ] Fazer GEM-X processar um vídeo curto e medir tempo
3. [ ] Estruturar o exercises-dataset num formato consultável (JSON/SQLite)
4. [ ] Prototipar a camada de comparação (pose do usuário vs referência)
5. [ ] Montar o loop mínimo: câmera → esqueleto → feedback texto
6. [ ] Só depois: avatar 3D + mundo virtual + IA personalizada

**Métrica do plano de 90 dias:** uma coisa verificável por dia.
Dia 1 = trt_pose rodando com latência medida.

---

## 7. Decisões em Aberto

- [ ] IA personalizada: LLM remoto (Gemini/GPT) ou modelo local?
- [ ] Onde roda o pipeline: no celular do usuário ou num servidor?
- [ ] Monetização: freemium com exercícios básicos, pago com IA personalizada?
- [ ] Fitverso é app separado ou módulo do Tourverse?

---

*Arquivo gerado em 2026-10-25. Atualizar conforme os forks evoluírem.*