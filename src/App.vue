<template>
  <div class="app">

    <!-- ==================== 顶部导航 ==================== -->
    <header class="top-nav">
      <div class="nav-inner">
        <div
          v-for="item in navItems"
          :key="item.id"
          class="nav-item"
          @click="goTo(item.id)"
        >
          {{ item.name }}
        </div>
      </div>
    </header>


    <!-- ==================== 首页 ==================== -->
    <section v-if="currentPage === 'home'" class="home-page">

      <!-- Hero -->
      <section class="hero-section">

        <div class="hero-brush"></div>

        <img
          v-if="heroImageVisible"
          src="/hero.png"
          class="hero-image"
          alt="手写数字识别系统"
          @error="heroImageVisible = false"
        />

        <!-- 如果 hero.png 没找到，仍然显示一个漂亮的备用首页 -->
        <div v-if="!heroImageVisible" class="hero-fallback">
          <div class="hero-en">
            HANDWRITTEN DIGIT
            <br />
            RECOGNITION SYSTEM
          </div>

          <div class="hero-cn">
            手写数字识别系统
          </div>

          <div class="hero-subtitle">
            「数字栖于墨痕，认知源于算法」
          </div>

          <div class="hero-small">
            基于深度学习的复杂场景手写数字智能识别与分析系统
          </div>
        </div>

      </section>


      <!-- 首页功能区域 -->
      <section class="home-content">

        <!-- 左侧按钮 -->
        <div class="menu-area">

          <button
            v-for="item in menuItems"
            :key="item.id"
            class="feature-button"
            @click="goTo(item.id)"
          >
            <span class="feature-icon">
              {{ item.icon }}
            </span>

            <span class="feature-text">
              {{ item.name }}
            </span>

            <span class="feature-arrow">
              →
            </span>
          </button>

        </div>


        <!-- 右侧系统概述 -->
        <div class="overview-box">

          <div class="overview-title">
            系统概述
          </div>

          <div class="overview-line"></div>

          <p>
            本系统是一个基于深度学习与前后端分离架构的手写数字智能识别平台。
          </p>

          <p>
            系统采用 PyTorch 构建卷积神经网络（CNN）模型，
            后端使用 Flask 提供 RESTful API，
            数据通过 SQLite 进行持久化管理，
            前端使用 Vue 3 构建交互式应用。
          </p>

          <p>
            在传统图片上传识别的基础上，
            系统进一步加入了
            <strong>手写画板、识别结果反馈、历史记录、数据统计以及用户体验反馈</strong>
            等功能。
          </p>

          <div class="tech-tags">
            <span>PyTorch</span>
            <span>Flask</span>
            <span>Vue 3</span>
            <span>SQLite</span>
            <span>OpenCV</span>
          </div>

        </div>

      </section>

    </section>


    <!-- ==================== 数字识别 ==================== -->
    <section v-else-if="currentPage === 'recognition'" class="page-container">

      <PageHeader
        title="手写数字识别"
        subtitle="图片上传 · 数字画板 · 智能识别 · 人工反馈"
      />


      <div class="recognition-grid">

        <!-- 左侧输入 -->
        <div class="panel">

          <div class="panel-title">
            选择识别方式
          </div>

          <div class="mode-switch">
            <button
              :class="{ active: recognitionMode === 'upload' }"
              @click="recognitionMode = 'upload'"
            >
              图片上传
            </button>

            <button
              :class="{ active: recognitionMode === 'canvas' }"
              @click="recognitionMode = 'canvas'"
            >
              手写画板
            </button>
          </div>


          <!-- 图片上传 -->
          <div v-if="recognitionMode === 'upload'" class="upload-area">

            <label class="upload-box">

              <input
                type="file"
                accept="image/*"
                @change="handleFileChange"
              />

              <div class="upload-icon">
                ＋
              </div>

              <div class="upload-title">
                点击选择图片
              </div>

              <div class="upload-desc">
                支持 JPG / PNG / BMP / WEBP
              </div>

            </label>

            <div v-if="selectedFileName" class="selected-file">
              已选择：{{ selectedFileName }}
            </div>

            <img
              v-if="previewUrl"
              :src="previewUrl"
              class="preview-image"
              alt="待识别图片"
            />

          </div>


          <!-- Canvas -->
          <div v-else class="canvas-area">

            <canvas
              ref="canvasRef"
              class="drawing-canvas"
              width="320"
              height="320"
              @mousedown="startDrawing"
              @mousemove="draw"
              @mouseup="stopDrawing"
              @mouseleave="stopDrawing"
              @touchstart.prevent="startTouchDrawing"
              @touchmove.prevent="touchDrawing"
              @touchend.prevent="stopDrawing"
            ></canvas>

            <div class="canvas-actions">

              <button
                class="primary-button"
                @click="clearCanvas"
              >
                清空画板
              </button>

              <button
                class="primary-button"
                @click="recognizeCanvas"
              >
                识别手写数字
              </button>

            </div>

          </div>


          <!-- 图片识别按钮 -->
          <button
            v-if="recognitionMode === 'upload'"
            class="primary-button full-button"
            :disabled="loading || !selectedFile"
            @click="recognizeImage"
          >
            {{ loading ? '正在识别……' : '开始识别' }}
          </button>

        </div>


        <!-- 右侧识别结果 -->
        <div class="panel result-panel">

          <div class="panel-title">
            识别结果
          </div>

          <div v-if="!predictionResult" class="empty-result">
            <div class="empty-symbol">

            </div>

            <p>
              请上传图片或在画板中书写数字
            </p>
          </div>


          <div v-else class="result-content">

            <div class="prediction-number">
              {{ predictionResult.prediction }}
            </div>

            <div class="prediction-label">
              模型预测结果
            </div>


            <div class="confidence-box">

              <div class="confidence-header">
                <span>识别置信度</span>

                <strong>
                  {{ formatPercent(predictionResult.confidence) }}
                </strong>
              </div>

              <div class="confidence-track">
                <div
                  class="confidence-value"
                  :style="{ width: (predictionResult.confidence * 100) + '%' }"
                ></div>
              </div>

              <div class="confidence-level">
                {{ getConfidenceText(predictionResult.confidence_level) }}
              </div>

            </div>


            <!-- Top 3 -->
            <div class="top3-box">

              <div class="result-section-title">
                Top-3 候选结果
              </div>

              <div
                v-for="(item, index) in predictionResult.top3"
                :key="index"
                class="top3-item"
              >

                <span class="rank">
                  {{ index + 1 }}
                </span>

                <span class="digit">
                  {{ item.digit }}
                </span>

                <div class="top3-bar">
                  <div
                    class="top3-fill"
                    :style="{ width: (item.probability * 100) + '%' }"
                  ></div>
                </div>

                <span class="probability">
                  {{ formatPercent(item.probability) }}
                </span>

              </div>

            </div>


            <!-- 智能建议 -->
            <div class="suggestion-box">

              <div class="result-section-title">
                识别建议
              </div>

              <div
                v-for="(suggestion, index) in predictionResult.suggestions"
                :key="index"
                class="suggestion-item"
              >
                <span>✓</span>
                {{ suggestion }}
              </div>

            </div>


            <!-- 结果反馈 -->
            <div
              v-if="predictionResult.history_id"
              class="feedback-box"
            >

              <div class="result-section-title">
                本次识别是否正确？
              </div>

              <div v-if="!feedbackSubmitted">

                <div class="feedback-buttons">

                  <button
                    class="correct-button"
                    @click="submitCorrect"
                  >
                    ✓ 正确
                  </button>

                  <button
                    class="wrong-button"
                    @click="showCorrection = true"
                  >
                    ✕ 不正确
                  </button>

                </div>


                <!-- 修正数字 -->
                <div
                  v-if="showCorrection"
                  class="correction-area"
                >

                  <div class="correction-title">
                    请告诉我们正确答案
                  </div>

                  <div class="digit-selector">

                    <button
                      v-for="digit in 10"
                      :key="digit - 1"
                      :class="{
                        selected: correctedDigit === digit - 1
                      }"
                      @click="correctedDigit = digit - 1"
                    >
                      {{ digit - 1 }}
                    </button>

                  </div>

                  <button
                    class="primary-button"
                    :disabled="correctedDigit === null"
                    @click="submitCorrection"
                  >
                    提交修正
                  </button>

                </div>

              </div>


              <div
                v-else
                class="thanks-message"
              >
                ✓ 感谢你的反馈！

                <br />

                你的修正已经成功记录，
                将帮助我们持续改进识别系统。
              </div>

            </div>

          </div>

        </div>

      </div>

    </section>


    <!-- ==================== 操作说明 ==================== -->
    <section
      v-else-if="currentPage === 'guide'"
      class="page-container"
    >

      <PageHeader
        title="操作说明与功能介绍"
        subtitle="了解系统的完整使用流程与核心功能"
      />


      <div class="guide-grid">

        <div class="info-card">

          <div class="info-number">
            01
          </div>

          <h3>
            手写数字识别
          </h3>

          <p>
            用户可以上传手机拍摄的手写数字图片，
            也可以直接在网页画板中书写数字。
            系统经过图像预处理后交由 PyTorch 深度学习模型进行识别。
          </p>

        </div>


        <div class="info-card">

          <div class="info-number">
            02
          </div>

          <h3>
            智能图像预处理
          </h3>

          <p>
            系统结合灰度化、背景处理、自适应阈值、
            Otsu 等方法提取数字区域，
            并将不同尺寸的数字统一转换为模型所需要的输入格式。
          </p>

        </div>


        <div class="info-card">

          <div class="info-number">
            03
          </div>

          <h3>
            置信度与 Top-3
          </h3>

          <p>
            模型不仅输出最终预测数字，
            同时返回识别置信度和 Top-3 候选结果，
            帮助用户判断当前结果是否可靠。
          </p>

        </div>


        <div class="info-card">

          <div class="info-number">
            04
          </div>

          <h3>
            失败反馈机制
          </h3>

          <p>
            用户可以对识别结果进行人工确认。
            如果模型识别错误，可以直接选择正确数字，
            系统将保存模型预测结果与用户修正结果。
          </p>

        </div>


        <div class="info-card">

          <div class="info-number">
            05
          </div>

          <h3>
            历史记录查询
          </h3>

          <p>
            每次识别都会自动保存识别时间、预测结果、
            置信度、Top-3 候选以及用户反馈，
            形成完整的识别记录。
          </p>

        </div>


        <div class="info-card">

          <div class="info-number">
            06
          </div>

          <h3>
            数据可视化
          </h3>

          <p>
            系统根据历史识别数据统计各数字出现频率，
            并通过柱状图直观展示，
            帮助用户了解自己的识别习惯。
          </p>

        </div>

      </div>


      <div class="process-card">

        <div class="process-title">
          系统使用流程
        </div>

        <div class="process">

          <div class="process-step">
            <span>01</span>
            上传 / 书写
          </div>

          <div class="process-arrow">
            →
          </div>

          <div class="process-step">
            <span>02</span>
            图像预处理
          </div>

          <div class="process-arrow">
            →
          </div>

          <div class="process-step">
            <span>03</span>
            CNN 模型识别
          </div>

          <div class="process-arrow">
            →
          </div>

          <div class="process-step">
            <span>04</span>
            结果反馈
          </div>

          <div class="process-arrow">
            →
          </div>

          <div class="process-step">
            <span>05</span>
            数据统计
          </div>

        </div>

      </div>

    </section>


    <!-- ==================== 历史记录 ==================== -->
    <section
      v-else-if="currentPage === 'history'"
      class="page-container"
    >

      <PageHeader
        title="历史记录查询"
        subtitle="查看系统保存的全部识别记录"
      />


      <div class="history-toolbar">

        <div>
          共
          <strong>{{ historyList.length }}</strong>
          条识别记录
        </div>

        <button
          class="danger-button"
          :disabled="historyList.length === 0"
          @click="clearHistory"
        >
          清空历史记录
        </button>

      </div>


      <div
        v-if="historyList.length === 0"
        class="empty-page"
      >
        暂无历史识别记录
      </div>


      <div
        v-else
        class="history-table-wrapper"
      >

        <table class="history-table">

          <thead>
            <tr>
              <th>图片</th>
              <th>识别时间</th>
              <th>模型结果</th>
              <th>置信度</th>
              <th>用户反馈</th>
              <th>修正结果</th>
              <th>操作</th>
            </tr>
          </thead>

          <tbody>

            <tr
              v-for="item in historyList"
              :key="item.id"
            >

              <td>

                <img
                  v-if="item.image_url"
                  :src="getImageUrl(item.image_url)"
                  class="history-image"
                  alt="识别图片"
                />

              </td>

              <td>
                {{ item.created_at }}
              </td>

              <td class="history-prediction">
                {{ item.prediction }}
              </td>

              <td>
                {{ formatPercent(item.confidence) }}
              </td>

              <td>

                <span
                  v-if="item.user_correct === 1"
                  class="status correct"
                >
                  ✓ 正确
                </span>

                <span
                  v-else-if="item.user_correct === 0"
                  class="status wrong"
                >
                  ✕ 错误
                </span>

                <span
                  v-else
                  class="status pending"
                >
                  待确认
                </span>

              </td>

              <td>
                <span v-if="item.corrected_prediction !== null">
                  {{ item.corrected_prediction }}
                </span>

                <span v-else>
                  —
                </span>
              </td>

              <td>

                <button
                  class="small-danger"
                  @click="deleteHistory(item.id)"
                >
                  删除
                </button>

              </td>

            </tr>

          </tbody>

        </table>

      </div>

    </section>


    <!-- ==================== 我的识别数据 ==================== -->
    <section
      v-else-if="currentPage === 'stats'"
      class="page-container"
    >

      <PageHeader
        title="我的识别数据"
        subtitle="从历史识别记录中了解自己的使用情况"
      />


      <div class="stats-grid">

        <div class="stat-card">
          <div class="stat-label">
            累积识别
          </div>

          <div><b>
            {{ stats.total }}
          </b></div>

          <div class="stat-unit">
            次
          </div>
        </div>


        <div class="stat-card">
          <div class="stat-label">
            准确确认
          </div>

          <div><b>
            {{ stats.correct }}
          </b></div>

          <div class="stat-unit">
            次
          </div>
        </div>


        <div class="stat-card">
          <div class="stat-label">
            待确认
          </div>

          <div><b>
            {{ stats.pending }}
          </b></div>

          <div class="stat-unit">
            次
          </div>
        </div>


        <div class="stat-card">
          <div class="stat-label">
            低置信度
          </div>

          <div><b>
            {{ stats.low_confidence }}
          </b></div>

          <div class="stat-unit">
            次
          </div>
        </div>

      </div>


      <div class="chart-card">

        <div class="chart-header">

          <div>
            <div class="chart-title">
              数字识别频率
            </div>

            <div class="chart-subtitle">
              根据历史识别结果统计
            </div>
          </div>

        </div>


        <div
          v-if="stats.digit_counts.length === 0"
          class="empty-page"
        >
          暂无足够的数据进行统计
        </div>


        <div
          v-else
          class="bar-chart"
        >

          <div
            v-for="digit in digitChartData"
            :key="digit.digit"
            class="bar-column"
          >

            <div class="bar-count">
              {{ digit.count }}
            </div>

            <div class="bar-area">

              <div
                class="bar"
                :style="{
                  height: getBarHeight(digit.count) + '%'
                }"
              ></div>

            </div>

            <div class="bar-label">
              {{ digit.digit }}
            </div>

          </div>

        </div>

      </div>


      <div
        v-if="mostRecognized"
        class="most-card"
      >

        <div>
          最常识别数字
        </div>

        <strong>
          {{ mostRecognized.digit }}
        </strong>

        <span>
          共 {{ mostRecognized.count }} 次
        </span>

      </div>

    </section>


    <!-- ==================== 用户反馈 ==================== -->
    <section
      v-else-if="currentPage === 'feedback'"
      class="page-container"
    >

      <PageHeader
        title="用户体验反馈箱"
        subtitle="你的每一条建议，都会帮助系统继续改进"
      />


      <div class="feedback-page-card">

        <div class="feedback-intro">

          <div class="feedback-big-icon">
            ✦
          </div>

          <div>

            <h2>
              告诉我们你的使用体验
            </h2>

            <p>
              如果在使用过程中遇到问题，
              或者有更好的功能建议，都欢迎告诉我们。
            </p>

          </div>

        </div>


        <div class="feedback-question">
          你认为系统目前有哪些方面需要改进？
        </div>


        <div class="feedback-options">

          <button
            v-for="item in experienceOptions"
            :key="item"
            :class="{
              selected: selectedExperience.includes(item)
            }"
            @click="toggleExperience(item)"
          >
            {{ item }}
          </button>

        </div>


        <div class="feedback-question">
          其他建议
        </div>

        <textarea
          v-model="experienceSuggestion"
          class="feedback-textarea"
          placeholder="可以告诉我们你希望增加什么功能，或者哪里使用起来不方便……"
        ></textarea>


        <button
          class="primary-button feedback-submit"
          :disabled="feedbackLoading"
          @click="submitExperienceFeedback"
        >
          {{ feedbackLoading ? '正在提交……' : '提交反馈' }}
        </button>


        <div
          v-if="experienceSuccess"
          class="feedback-success"
        >
          ✓ 感谢你的反馈！

          <br />

          我们已经收到你的建议。
        </div>

      </div>

    </section>


    <!-- ==================== 返回首页 ==================== -->
    <button
      v-if="currentPage !== 'home'"
      class="back-home"
      @click="goTo('home')"
    >
      ← 返回首页
    </button>


    <!-- ==================== Footer ==================== -->
    <footer
      v-if="currentPage === 'home'"
      class="footer"
    >
      <div>
        HANDWRITTEN DIGIT RECOGNITION SYSTEM
      </div>

      <span>
        PyTorch · Flask · Vue 3 · OpenCV · SQLite
      </span>
    </footer>

  </div>
</template>


<script setup>

import {
  ref,
  computed,
  onMounted,
  nextTick
} from 'vue'


/* =========================================================
   基础配置
========================================================= */

const API_BASE = 'http://127.0.0.1:5000'


/* =========================================================
   页面
========================================================= */

const currentPage = ref('home')

const navItems = [
  {
    id: 'feedback',
    name: '用户反馈闭环'
  },
  {
    id: 'recognition',
    name: '数字智能识别'
  },
  {
    id: 'guide',
    name: '操作说明'
  },
  {
    id: 'history',
    name: '历史记录查询'
  },
  {
    id: 'stats',
    name: '数据可视化'
  }
]

const menuItems = [
  {
    id: 'recognition',
    name: '手写数字识别',
    icon: '⌕'
  },
  {
    id: 'guide',
    name: '操作说明与功能介绍',
    icon: '↗'
  },
  {
    id: 'history',
    name: '历史记录查询',
    icon: '▦'
  },
  {
    id: 'stats',
    name: '我的识别数据',
    icon: '⌁'
  },
  {
    id: 'feedback',
    name: '用户体验反馈箱',
    icon: '▱'
  }
]


function goTo(page) {

  currentPage.value = page

  window.scrollTo({
    top: 0,
    behavior: 'smooth'
  })

  if (page === 'history') {
    loadHistory()
  }

  if (page === 'stats') {
    loadStats()
  }

  if (page === 'recognition') {
    nextTick(() => {
      if (recognitionMode.value === 'canvas') {
        initCanvas()
      }
    })
  }

}


/* =========================================================
   Hero
========================================================= */

const heroImageVisible = ref(true)


/* =========================================================
   图片识别
========================================================= */

const recognitionMode = ref('upload')

const selectedFile = ref(null)

const selectedFileName = ref('')

const previewUrl = ref('')

const loading = ref(false)

const predictionResult = ref(null)


function handleFileChange(event) {

  const file = event.target.files[0]

  if (!file) {
    return
  }

  selectedFile.value = file

  selectedFileName.value = file.name

  if (previewUrl.value) {
    URL.revokeObjectURL(previewUrl.value)
  }

  previewUrl.value = URL.createObjectURL(file)

  predictionResult.value = null

}


async function recognizeImage() {

  if (!selectedFile.value) {
    alert('请先选择一张图片')
    return
  }

  loading.value = true

  predictionResult.value = null

  const formData = new FormData()

  formData.append(
    'image',
    selectedFile.value
  )

  try {

    const response = await fetch(
      `${API_BASE}/predict`,
      {
        method: 'POST',
        body: formData
      }
    )

    const data = await response.json()

    if (!response.ok || !data.success) {
      throw new Error(
        data.message || '识别失败'
      )
    }

    predictionResult.value = data

  } catch (error) {

    console.error(error)

    alert(
      `识别失败：${error.message}`
    )

  } finally {

    loading.value = false

  }

}


/* =========================================================
   Canvas
========================================================= */

const canvasRef = ref(null)

const drawing = ref(false)

const lastPoint = ref({
  x: 0,
  y: 0
})


function initCanvas() {

  const canvas = canvasRef.value

  if (!canvas) {
    return
  }

  const ctx = canvas.getContext('2d')

  ctx.fillStyle = '#ffffff'

  ctx.fillRect(
    0,
    0,
    canvas.width,
    canvas.height
  )

  ctx.strokeStyle = '#111111'

  ctx.lineWidth = 18

  ctx.lineCap = 'round'

  ctx.lineJoin = 'round'

}


function getCanvasPoint(event) {

  const canvas = canvasRef.value

  const rect = canvas.getBoundingClientRect()

  return {
    x:
      (event.clientX - rect.left)
      *
      canvas.width
      /
      rect.width,

    y:
      (event.clientY - rect.top)
      *
      canvas.height
      /
      rect.height
  }

}


function startDrawing(event) {

  const point = getCanvasPoint(event)

  drawing.value = true

  lastPoint.value = point

}


function draw(event) {

  if (!drawing.value) {
    return
  }

  const canvas = canvasRef.value

  const ctx = canvas.getContext('2d')

  const point = getCanvasPoint(event)

  ctx.beginPath()

  ctx.moveTo(
    lastPoint.value.x,
    lastPoint.value.y
  )

  ctx.lineTo(
    point.x,
    point.y
  )

  ctx.stroke()

  lastPoint.value = point

}


function stopDrawing() {

  drawing.value = false

}


function startTouchDrawing(event) {

  const touch = event.touches[0]

  if (!touch) {
    return
  }

  const point = getTouchPoint(touch)

  drawing.value = true

  lastPoint.value = point

}


function getTouchPoint(touch) {

  const canvas = canvasRef.value

  const rect = canvas.getBoundingClientRect()

  return {
    x:
      (touch.clientX - rect.left)
      *
      canvas.width
      /
      rect.width,

    y:
      (touch.clientY - rect.top)
      *
      canvas.height
      /
      rect.height
  }

}


function touchDrawing(event) {

  if (!drawing.value) {
    return
  }

  const touch = event.touches[0]

  if (!touch) {
    return
  }

  const canvas = canvasRef.value

  const ctx = canvas.getContext('2d')

  const point = getTouchPoint(touch)

  ctx.beginPath()

  ctx.moveTo(
    lastPoint.value.x,
    lastPoint.value.y
  )

  ctx.lineTo(
    point.x,
    point.y
  )

  ctx.stroke()

  lastPoint.value = point

}


function clearCanvas() {

  const canvas = canvasRef.value

  if (!canvas) {
    return
  }

  const ctx = canvas.getContext('2d')

  ctx.fillStyle = '#ffffff'

  ctx.fillRect(
    0,
    0,
    canvas.width,
    canvas.height
  )

}


async function recognizeCanvas() {

  const canvas = canvasRef.value

  if (!canvas) {
    return
  }

  const blob = await new Promise(
    resolve => canvas.toBlob(resolve, 'image/png')
  )

  if (!blob) {
    alert('无法获取画板图片')
    return
  }

  const file = new File(
    [blob],
    'canvas_digit.png',
    {
      type: 'image/png'
    }
  )

  selectedFile.value = file

  selectedFileName.value = '画板手写数字'

  await recognizeImage()

}


/* =========================================================
   识别反馈
========================================================= */

const showCorrection = ref(false)

const correctedDigit = ref(null)

const feedbackSubmitted = ref(false)


async function submitCorrect() {

  if (!predictionResult.value) {
    return
  }

  try {

    const response = await fetch(
      `${API_BASE}/feedback`,
      {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          history_id:
            predictionResult.value.history_id,

          is_correct: true
        })
      }
    )

    const data = await response.json()

    if (!response.ok || !data.success) {
      throw new Error(
        data.message || '反馈提交失败'
      )
    }

    feedbackSubmitted.value = true

  } catch (error) {

    alert(
      `反馈提交失败：${error.message}`
    )

  }

}


async function submitCorrection() {

  if (
    !predictionResult.value ||
    correctedDigit.value === null
  ) {
    return
  }

  try {

    const response = await fetch(
      `${API_BASE}/feedback`,
      {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({

          history_id:
            predictionResult.value.history_id,

          is_correct: false,

          corrected_digit:
            correctedDigit.value

        })
      }
    )

    const data = await response.json()

    if (!response.ok || !data.success) {
      throw new Error(
        data.message || '反馈提交失败'
      )
    }

    feedbackSubmitted.value = true

  } catch (error) {

    alert(
      `反馈提交失败：${error.message}`
    )

  }

}


/* =========================================================
   历史记录
========================================================= */

const historyList = ref([])


async function loadHistory() {

  try {

    const response = await fetch(
      `${API_BASE}/history`
    )

    const data = await response.json()

    if (!response.ok || !data.success) {
      throw new Error(
        data.message || '历史记录获取失败'
      )
    }

    historyList.value =
      data.data || []

  } catch (error) {

    console.error(error)

    alert(
      `历史记录加载失败：${error.message}`
    )

  }

}


async function deleteHistory(id) {

  if (
    !confirm('确定删除这条历史记录吗？')
  ) {
    return
  }

  try {

    const response = await fetch(
      `${API_BASE}/history/${id}`,
      {
        method: 'DELETE'
      }
    )

    const data = await response.json()

    if (!response.ok || !data.success) {
      throw new Error(
        data.message || '删除失败'
      )
    }

    await loadHistory()

  } catch (error) {

    alert(
      `删除失败：${error.message}`
    )

  }

}


async function clearHistory() {

  if (historyList.value.length === 0) {
    return
  }

  if (
    !confirm(
      '确定清空全部历史记录吗？此操作不可恢复。'
    )
  ) {
    return
  }

  try {

    const response = await fetch(
      `${API_BASE}/history`,
      {
        method: 'DELETE'
      }
    )

    const data = await response.json()

    if (!response.ok || !data.success) {
      throw new Error(
        data.message || '清空失败'
      )
    }

    historyList.value = []

  } catch (error) {

    alert(
      `清空失败：${error.message}`
    )

  }

}


/* =========================================================
   统计数据
========================================================= */

const stats = ref({
  total: 0,
  correct: 0,
  pending: 0,
  low_confidence: 0,
  digit_counts: []
})


async function loadStats() {

  try {

    const response = await fetch(
      `${API_BASE}/stats`
    )

    const data = await response.json()

    if (!response.ok || !data.success) {
      throw new Error(
        data.message || '统计数据获取失败'
      )
    }

    stats.value =
      data.data || stats.value

  } catch (error) {

    console.error(error)

    alert(
      `统计数据加载失败：${error.message}`
    )

  }

}


const digitChartData = computed(() => {

  const original =
    stats.value.digit_counts || []

  const map = {}

  for (let i = 0; i < 10; i++) {
    map[i] = 0
  }

  original.forEach(item => {

    map[item.digit] =
      Number(item.count) || 0

  })

  return Array.from(
    { length: 10 },
    (_, digit) => ({
      digit,
      count: map[digit]
    })
  )

})


const maxDigitCount = computed(() => {

  const values =
    digitChartData.value.map(
      item => item.count
    )

  return Math.max(
    1,
    ...values
  )

})


function getBarHeight(count) {

  if (!count) {
    return 3
  }

  return Math.max(
    5,
    (count / maxDigitCount.value) * 100
  )

}


const mostRecognized = computed(() => {

  const data =
    stats.value.digit_counts || []

  if (data.length === 0) {
    return null
  }

  return data[0]

})


/* =========================================================
   用户体验反馈
========================================================= */

const experienceOptions = [
  '识别结果较差',
  '图片上传不方便',
  '画板操作不方便',
  '识别速度较慢',
  '历史记录功能',
  '数据统计功能',
  '希望增加更多数字类型',
  '界面设计建议'
]


const selectedExperience = ref([])

const experienceSuggestion = ref('')

const feedbackLoading = ref(false)

const experienceSuccess = ref(false)


function toggleExperience(item) {

  const index =
    selectedExperience.value.indexOf(item)

  if (index === -1) {

    selectedExperience.value.push(item)

  } else {

    selectedExperience.value.splice(
      index,
      1
    )

  }

}


async function submitExperienceFeedback() {

  if (
    selectedExperience.value.length === 0 &&
    !experienceSuggestion.value.trim()
  ) {

    alert(
      '请选择反馈类型或填写你的建议'
    )

    return

  }

  feedbackLoading.value = true

  try {

    const response = await fetch(
      `${API_BASE}/experience-feedback`,
      {
        method: 'POST',

        headers: {
          'Content-Type': 'application/json'
        },

        body: JSON.stringify({

          types:
            selectedExperience.value,

          suggestion:
            experienceSuggestion.value

        })

      }
    )

    const data =
      await response.json()

    if (!response.ok || !data.success) {

      throw new Error(
        data.message || '反馈提交失败'
      )

    }

    experienceSuccess.value = true

    selectedExperience.value = []

    experienceSuggestion.value = ''

  } catch (error) {

    alert(
      `反馈提交失败：${error.message}`
    )

  } finally {

    feedbackLoading.value = false

  }

}


/* =========================================================
   工具
========================================================= */

function formatPercent(value) {

  if (
    value === undefined ||
    value === null
  ) {
    return '—'
  }

  return (
    Number(value) * 100
  ).toFixed(2) + '%'

}


function getConfidenceText(level) {

  if (level === 'HIGH') {
    return '高置信度'
  }

  if (level === 'MEDIUM') {
    return '中等置信度'
  }

  return '低置信度'

}


function getImageUrl(url) {

  if (!url) {
    return ''
  }

  if (
    url.startsWith('http://') ||
    url.startsWith('https://')
  ) {
    return url
  }

  return API_BASE + url

}


/* =========================================================
   页面初始化
========================================================= */

onMounted(() => {

  initCanvas()

})

</script>


<!-- =========================================================
     子组件：页面标题
========================================================= -->

<script>
export default {
  components: {
    PageHeader: {
      props: {
        title: {
          type: String,
          required: true
        },
        subtitle: {
          type: String,
          default: ''
        }
      },

      template: `
        <div class="page-header">
          <div class="page-header-en">
            HANDWRITTEN DIGIT
          </div>

          <h1>
            {{ title }}
          </h1>

          <p>
            {{ subtitle }}
          </p>

          <div class="header-line"></div>
        </div>
      `
    }
  }
}
</script>


<style>

@import url(
  'https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@500;600&family=Noto+Serif+SC:wght@400;500;600;700&display=swap'
);


/* =========================================================
   全局
========================================================= */

:root {

  --ink: #292725;

  --ink-light: #55504b;

  --paper: #f1f0ec;

  --paper-dark: #e4e2dc;

  --white: #ffffff;

  --line: #cfcac2;

  --accent: #74635a;

  --accent-light: #a9988c;

}


* {
  box-sizing: border-box;
}


html {
  scroll-behavior: smooth;
}


body {

  margin: 0;

  background:
    radial-gradient(
      circle at 10% 20%,
      rgba(80, 75, 68, 0.035),
      transparent 30%
    ),
    radial-gradient(
      circle at 90% 70%,
      rgba(80, 75, 68, 0.04),
      transparent 35%
    ),
    #DDDDDD;

  color: var(--ink);

  font-family:
    "Noto Serif SC",
    "Microsoft YaHei",
    serif;

}


button,
input,
textarea {
  font-family: inherit;
}


/* =========================================================
   App
========================================================= */

.app {

  min-height: 100vh;

  position: relative;

  overflow-x: hidden;

}


.app::before {

  content: "";

  position: fixed;

  inset: 0;

  pointer-events: none;

  opacity: 0.18;

  background:
    linear-gradient(
      100deg,
      transparent 0%,
      rgba(110, 103, 96, 0.08) 20%,
      transparent 40%,
      rgba(110, 103, 96, 0.06) 65%,
      transparent 90%
    );

  z-index: 0;

}


/* =========================================================
   顶部导航
========================================================= */

.top-nav {

  position: relative;

  z-index: 10;

  width: 100%;

  background:#C9C9C9;

  border-bottom: 1px solid
    rgba(50, 48, 45, 0.1);

}


.nav-inner {

  width: min(
    1180px,
    calc(100% - 48px)
  );

  margin: 0 auto;

  min-height: 64px;

  display: flex;

  align-items: center;

  justify-content: center;

  gap: 72px;

}


.nav-item {

  font-size: 15px;

  letter-spacing: 1px;

  cursor: pointer;

  color: #393633;

  position: relative;

  transition: 0.25s;

}


.nav-item::after {

  content: "";

  position: absolute;

  left: 50%;

  bottom: -9px;

  width: 0;

  height: 1px;

  background: var(--ink);

  transform: translateX(-50%);

  transition: 0.25s;

}


.nav-item:hover {

  color: #000;

}


.nav-item:hover::after {

  width: 100%;

}


/* =========================================================
   首页 Hero
========================================================= */
.hero-section {
  /* 1. 宽度改为 100%，取消两边留白，与导航栏完全对齐 */
  width: 100%;
  /* 2. 去掉顶部边距和左右自动边距 */
  margin: 0;
  /* 3. 增加高度限制，让图片放得更大（原来最大只有 620px，现在放大约 1.5 倍） */
  height: clamp(500px, 65vh, 850px);
  max-height: 850px; /* 同步增加最大高度限制 */
  position: relative;
  overflow: hidden;
  /* 4. 去掉圆角，让背景与导航栏自然相接 */
  border-radius: 0;
  background: #faf9f6;
}


.hero-section::after {

  content: "";

  position: absolute;

  inset: 0;

  background:
    linear-gradient(
      to bottom,
      rgba(255,255,255,0.03),
      rgba(255,255,255,0.12)
    );

  pointer-events: none;

}


.hero-image {

  width: 100%;

  height: 100%;

  object-fit: contain;

  object-position: center;

  display: block;

}


.hero-brush {

  position: absolute;

  inset: 0;

  background:
    linear-gradient(
      115deg,
      transparent 5%,
      rgba(90,85,80,0.06) 16%,
      transparent 30%
    ),
    linear-gradient(
      75deg,
      transparent 40%,
      rgba(90,85,80,0.055) 53%,
      transparent 68%
    );

  pointer-events: none;

}


.hero-fallback {

  position: absolute;

  inset: 0;

  display: flex;

  flex-direction: column;

  align-items: center;

  justify-content: center;

  text-align: center;

}


.hero-en {

  font-family:
    "Cormorant Garamond",
    serif;

  font-size: clamp(
    36px,
    5vw,
    72px
  );

  line-height: 1.05;

  letter-spacing: 5px;

}


.hero-cn {

  margin-top: 35px;

  font-size: clamp(
    42px,
    6vw,
    84px
  );

  letter-spacing: 18px;

}


.hero-subtitle {

  margin-top: 18px;

  font-size: 24px;

  letter-spacing: 5px;

  color: #76655d;

}


.hero-small {

  margin-top: 65px;

  color: #66605a;

  font-size: 14px;

  letter-spacing: 2px;

}


/* =========================================================
   首页功能区域
========================================================= */

.home-content {

  width: min(
    1180px,
    calc(100% - 48px)
  );

  margin: 55px auto 90px;

  display: grid;

  grid-template-columns:
    minmax(0, 1.15fr)
    minmax(360px, 0.85fr);

  gap: 55px;

  align-items: center;

}


.menu-area {

  display: flex;

  flex-direction: column;

  gap: 24px;

}


.feature-button {

  min-height: 76px;

  width: 100%;

  border: 1px solid #35322f;

  border-radius: 40px;

  background:
    rgba(
      255,
      255,
      255,
      0.72
    );

  display: grid;

  grid-template-columns:
    68px 1fr 50px;

  align-items: center;

  text-align: left;

  padding: 0 25px;

  cursor: pointer;

  transition:
    transform 0.25s,
    background 0.25s,
    box-shadow 0.25s;

}


.feature-button:hover {

  transform:
    translateX(8px);

  background: #fff;

  box-shadow:
    0 10px 30px
    rgba(40, 38, 35, 0.08);

}


.feature-icon {

  width: 48px;

  height: 48px;

  border-radius: 50%;

  display: flex;

  align-items: center;

  justify-content: center;

  background: #302e2c;

  color: #fff;

  font-size: 25px;

}


.feature-text {

  font-size: 24px;

  letter-spacing: 2px;

}


.feature-arrow {

  font-size: 24px;

  text-align: right;

  opacity: 0.55;

}


/* =========================================================
   系统概述
========================================================= */

.overview-box {

  background:
    #C9C9C9;

  padding: 48px;

  min-height: 390px;

  display: flex;

  flex-direction: column;

  justify-content: center;

}


.overview-title {

  font-size: 32px;

  letter-spacing: 5px;

}


.overview-line {

  width: 350px;

  height: 2px;

  background: #47423e;

  margin: 22px 0 25px;

}


.overview-box p {

  font-size: 16px;

  line-height: 2;

  color: #494541;

  margin: 0 0 15px;

}


.tech-tags {

  display: flex;

  flex-wrap: wrap;

  gap: 10px;

  margin-top: 20px;

}


.tech-tags span {

  border: 1px solid #aaa49b;

  border-radius: 20px;

  padding: 7px 15px;

  color: #655e57;

  font-size: 13px;

}


/* =========================================================
   页面统一容器
========================================================= */

.page-container {

  width: min(
    1180px,
    calc(100% - 48px)
  );

  margin: 0 auto;

  padding:
    70px
    0
    100px;

  position: relative;

  z-index: 1;

}


.page-header {

  text-align: center;

  margin-bottom: 55px;

}


.page-header-en {

  font-family:
    "Cormorant Garamond",
    serif;

  letter-spacing: 5px;

  font-size: 14px;

  color: #81786f;

  margin-bottom: 8px;

}


.page-header h1 {

  font-size: 42px;

  letter-spacing: 8px;

  margin: 0;

  font-weight: 500;

}


.page-header p {

  color: #77716a;

  margin-top: 15px;

}


.header-line {

  width: 80px;

  height: 1px;

  background: #4b4743;

  margin: 25px auto 0;

}


/* =========================================================
   通用 Panel
========================================================= */

.panel {

  background:
    rgba(
      255,
      255,
      255,
      0.82
    );

  border: 1px solid
    rgba(50,48,45,0.1);

  padding: 35px;

  border-radius: 4px;

  box-shadow:
    0 15px 40px
    rgba(40, 38, 35, 0.055);

}


.panel-title {

  font-size: 23px;

  letter-spacing: 3px;

  margin-bottom: 25px;

}


/* =========================================================
   识别区域
========================================================= */

.recognition-grid {

  display: grid;

  grid-template-columns:
    minmax(0, 1fr)
    minmax(0, 1fr);

  gap: 35px;

}


.mode-switch {

  display: flex;

  gap: 10px;

  margin-bottom: 25px;

}


.mode-switch button {

  flex: 1;

  padding: 13px;

  border: 1px solid #c6c0b8;

  background: #f6f4ef;

  cursor: pointer;

}


.mode-switch button.active {

  background: #302e2c;

  color: white;

  border-color: #302e2c;

}


/* =========================================================
   上传
========================================================= */

.upload-box {

  border: 1px dashed #a9a39a;

  min-height: 220px;

  display: flex;

  flex-direction: column;

  align-items: center;

  justify-content: center;

  cursor: pointer;

  background: #faf9f6;

}


.upload-box input {

  display: none;

}


.upload-icon {

  width: 55px;

  height: 55px;

  border-radius: 50%;

  background: #35322f;

  color: white;

  display: flex;

  align-items: center;

  justify-content: center;

  font-size: 30px;

}


.upload-title {

  margin-top: 18px;

  font-size: 19px;

}


.upload-desc {

  margin-top: 8px;

  font-size: 13px;

  color: #8a847d;

}


.selected-file {

  margin-top: 12px;

  text-align: center;

  color: #766a61;

}


.preview-image {

  display: block;

  width: min(
    100%,
    320px
  );

  max-height: 260px;

  object-fit: contain;

  margin: 25px auto 0;

  border: 1px solid #ddd8d0;

}


/* =========================================================
   Canvas
========================================================= */

.canvas-area {

  display: flex;

  flex-direction: column;

  align-items: center;

}


.drawing-canvas {

  width: min(
    320px,
    100%
  );

  height: auto;

  background: white;

  border: 1px solid #aaa39a;

  cursor: crosshair;

  touch-action: none;

}


.canvas-actions {

  display: flex;

  gap: 12px;

  margin-top: 20px;

  width: 100%;

}


.canvas-actions button {

  flex: 1;

}


/* =========================================================
   Button
========================================================= */

.primary-button {

  margin-top: 20px;

  padding: 14px 28px;

  border: none;

  background: #302e2c;

  color: white;

  cursor: pointer;

  letter-spacing: 2px;

  transition: 0.25s;

}


.primary-button:hover {

  background: #514b46;

}


.primary-button:disabled {

  opacity: 0.4;

  cursor: not-allowed;

}


.secondary-button {

  padding: 14px 25px;

  border: 1px solid #77716a;

  background: transparent;

  cursor: pointer;

}


.full-button {

  width: 100%;

}


/* =========================================================
   结果
========================================================= */

.result-panel {

  min-height: 600px;

}


.empty-result {

  height: 480px;

  display: flex;

  flex-direction: column;

  align-items: center;

  justify-content: center;

  color: #8c867f;

}


.empty-symbol {

  font-family:
    "Cormorant Garamond",
    serif;

  font-size: 100px;

  color: #d0cbc3;

}


.prediction-number {

  text-align: center;

  font-size: 110px;

  line-height: 1;

  font-family:
    "Cormorant Garamond",
    serif;

}


.prediction-label {

  text-align: center;

  margin-top: 10px;

  color: #77716a;

}


.confidence-box {

  margin-top: 35px;

}


.confidence-header {

  display: flex;

  justify-content: space-between;

  align-items: center;

}


.confidence-header strong {

  font-size: 20px;

}


.confidence-track {

  height: 8px;

  background: #ddd9d2;

  margin-top: 10px;

  overflow: hidden;

}


.confidence-value {

  height: 100%;

  background: #514b46;

}


.confidence-level {

  margin-top: 8px;

  text-align: right;

  font-size: 13px;

  color: #77716a;

}


.result-section-title {

  font-size: 17px;

  margin-bottom: 15px;

}


.top3-box {

  margin-top: 35px;

}


.top3-item {

  display: grid;

  grid-template-columns:
    30px
    30px
    1fr
    65px;

  gap: 10px;

  align-items: center;

  margin-bottom: 12px;

}


.rank {

  color: #888;

}


.digit {

  font-weight: bold;

}


.top3-bar {

  height: 7px;

  background: #e5e1db;

}


.top3-fill {

  height: 100%;

  background: #6f6259;

}


.probability {

  text-align: right;

  font-size: 13px;

}


.suggestion-box {

  margin-top: 30px;

  padding: 20px;

  background: #f1efea;

}


.suggestion-item {

  margin: 8px 0;

  color: #5e5852;

}


.suggestion-item span {

  color: #5f7566;

  margin-right: 8px;

}


/* =========================================================
   Feedback
========================================================= */

.feedback-box {

  margin-top: 30px;

  padding-top: 25px;

  border-top: 1px solid #ddd8d0;

}


.feedback-buttons {

  display: flex;

  gap: 12px;

}


.correct-button,
.wrong-button {

  flex: 1;

  padding: 13px;

  border: none;

  cursor: pointer;

}


.correct-button {

  background: #e5eee7;

  color: #3e5c47;

}


.wrong-button {

  background: #eee8e3;

  color: #684f45;

}


.correction-area {

  margin-top: 20px;

}


.correction-title {

  margin-bottom: 12px;

}


.digit-selector {

  display: grid;

  grid-template-columns:
    repeat(10, 1fr);

  gap: 6px;

  margin-bottom: 10px;

}


.digit-selector button {

  aspect-ratio: 1;

  border: 1px solid #c5bfb7;

  background: #f7f5f0;

  cursor: pointer;

}


.digit-selector button.selected {

  background: #302e2c;

  color: white;

}


.thanks-message {

  padding: 20px;

  background: #edf2ed;

  color: #506354;

  line-height: 1.9;

}


/* =========================================================
   操作说明
========================================================= */

.guide-grid {

  display: grid;

  grid-template-columns:
    repeat(3, 1fr);

  gap: 22px;

}


.info-card {

  background: rgba(
    255,
    255,
    255,
    0.75
  );

  padding: 30px;

  border: 1px solid #ded9d2;

  min-height: 230px;

}


.info-number {

  font-family:
    serif;

  font-size: 38px;

  color: #aaa29a;

}


.info-card h3 {

  font-size: 21px;

  font-weight: 500;

}


.info-card p {

  color: #66615b;

  line-height: 1.9;

  font-size: 14px;

}


.process-card {

  margin-top: 35px;

  padding: 35px;

  background: #C6C6C6;

}


.process-title {

  font-size: 22px;

  margin-bottom: 25px;

}


.process {

  display: flex;

  align-items: center;

  justify-content: center;

  gap: 15px;

  flex-wrap: wrap;

}


.process-step {

  padding: 13px 18px;

  background: #f8f7f3;

}


.process-step span {

  margin-right: 7px;

  color: #8a8178;

}


.process-arrow {

  color: #8a8178;

}


/* =========================================================
   历史
========================================================= */

.history-toolbar {

  display: flex;

  justify-content: space-between;

  align-items: center;

  margin-bottom: 25px;

}


.danger-button,
.small-danger {

  border: 1px solid #987d72;

  background: transparent;

  color: #76574c;

  cursor: pointer;

}


.danger-button {

  padding: 11px 20px;

}


.danger-button:disabled {

  opacity: 0.4;

  cursor: not-allowed;

}


.history-table-wrapper {

  overflow-x: auto;

  background: white;

}


.history-table {

  width: 100%;

  border-collapse: collapse;

  min-width: 900px;

}


.history-table th {

  background: #e2dfd8;

  padding: 16px;

  text-align: center;

  font-weight: 500;

}


.history-table td {

  border-bottom: 1px solid #e2ded7;

  padding: 13px;

  text-align: center;

}


.history-image {

  width: 65px;

  height: 65px;

  object-fit: contain;

  border: 1px solid #ddd8d0;

}


.history-prediction {

  font-size: 25px;

}


.small-danger {

  padding: 7px 12px;

}


.status {

  padding: 5px 10px;

  border-radius: 20px;

  font-size: 12px;

}


.status.correct {

  background: #e3eee6;

  color: #46604e;

}


.status.wrong {

  background: #eee2df;

  color: #754e45;

}


.status.pending {

  background: #eeeae3;

  color: #776c5f;

}


/* =========================================================
   Stats
========================================================= */

.stats-grid {

  display: grid;

  grid-template-columns:
    repeat(4, 1fr);

  gap: 20px;

}


.stat-card {

  background: white;

  padding: 30px;

  text-align: center;

  border: 1px solid #ddd8d1;

}


.stat-label {

  color: #77716a;

}


.stat-number {

  font-family:
    "Cormorant Garamond",
    serif;

  font-size: 58px;

  margin-top: 12px;

}


.stat-unit {

  color: #99918a;

}


.chart-card {

  margin-top: 30px;

  background: white;

  padding: 35px;

  border: 1px solid #ddd8d1;

}


.chart-title {

  font-size: 23px;

}


.chart-subtitle {

  margin-top: 7px;

  color: #88817a;

  font-size: 13px;

}


.bar-chart {

  height: 350px;

  margin-top: 45px;

  display: flex;

  align-items: stretch;

  justify-content: space-around;

  gap: 15px;

}


.bar-column {

  flex: 1;

  display: flex;

  flex-direction: column;

  align-items: center;

  justify-content: flex-end;

}


.bar-count {

  margin-bottom: 8px;

  color: #625b54;

}


.bar-area {

  width: 100%;

  height: 270px;

  display: flex;

  align-items: flex-end;

  justify-content: center;

}


.bar {

  width: min(
    55px,
    65%
  );

  background: #6d625a;

  min-height: 4px;

  transition: height 0.4s;

}


.bar-label {

  margin-top: 12px;

  font-size: 18px;

}


.most-card {

  margin-top: 25px;

  background: #C9C9C9;

  padding: 25px;

  display: flex;

  align-items: center;

  gap: 20px;

}


.most-card strong {

  font-size: 45px;

  font-family:
    "Cormorant Garamond",
    serif;

}


.most-card span {

  color: #6d655e;

}


/* =========================================================
   用户反馈页面
========================================================= */

.feedback-page-card {

  max-width: 850px;

  margin: 0 auto;

  background: white;

  padding: 50px;

  border: 1px solid #ddd8d1;

}


.feedback-intro {

  display: flex;

  gap: 25px;

  align-items: center;

}


.feedback-big-icon {

  width: 65px;

  height: 65px;

  border-radius: 50%;

  background: #302e2c;

  color: white;

  display: flex;

  align-items: center;

  justify-content: center;

  font-size: 28px;

}


.feedback-intro h2 {

  margin: 0;

  font-size: 27px;

  font-weight: 500;

}


.feedback-intro p {

  color: #77716a;

}


.feedback-question {

  margin-top: 35px;

  margin-bottom: 15px;

  font-size: 18px;

}


.feedback-options {

  display: flex;

  flex-wrap: wrap;

  gap: 10px;

}


.feedback-options button {

  border: 1px solid #c8c1b8;

  background: #f8f6f1;

  padding: 11px 17px;

  cursor: pointer;

}


.feedback-options button.selected {

  background: #302e2c;

  color: white;

}


.feedback-textarea {

  width: 100%;

  min-height: 150px;

  resize: vertical;

  border: 1px solid #c9c3ba;

  background: #f8f7f3;

  padding: 15px;

  outline: none;

}


.feedback-textarea:focus {

  border-color: #69615a;

}


.feedback-submit {

  width: 100%;

}


.feedback-success {

  margin-top: 20px;

  background: #edf2ed;

  color: #4e6353;

  padding: 15px;

  line-height: 1.8;

}


/* =========================================================
   空状态
========================================================= */

.empty-page {

  padding: 100px 20px;

  text-align: center;

  color: #99918a;

  background: rgba(
    255,
    255,
    255,
    0.7
  );

}


/* =========================================================
   返回首页
========================================================= */

.back-home {

  position: fixed;

  right: 30px;

  bottom: 30px;

  z-index: 20;

  border: none;

  background: #302e2c;

  color: white;

  padding: 13px 20px;

  cursor: pointer;

  box-shadow:
    0 8px 25px
    rgba(0,0,0,0.15);

}


/* =========================================================
   Footer
========================================================= */

.footer {

  width: min(
    1180px,
    calc(100% - 48px)
  );

  margin: 0 auto;

  padding: 35px 0 50px;

  border-top: 1px solid #d8d3cb;

  text-align: center;

  color: #888078;

  font-family:
    "Cormorant Garamond",
    serif;

  letter-spacing: 3px;

}


.footer span {

  display: block;

  margin-top: 10px;

  font-family:
    "Noto Serif SC",
    serif;

  font-size: 12px;

  letter-spacing: 1px;

}


/* =========================================================
   响应式
========================================================= */

@media (max-width: 900px) {

  .nav-inner {

    gap: 25px;

  }


  .home-content {

    grid-template-columns: 1fr;

  }


  .recognition-grid {

    grid-template-columns: 1fr;

  }


  .guide-grid {

    grid-template-columns:
      repeat(2, 1fr);

  }


  .stats-grid {

    grid-template-columns:
      repeat(2, 1fr);

  }

}


@media (max-width: 600px) {

  .nav-inner {

    width: 100%;

    padding: 0 15px;

    gap: 15px;

    overflow-x: auto;

    justify-content: flex-start;

  }


  .nav-item {

    white-space: nowrap;

    font-size: 13px;

  }


  .hero-section {

    width: calc(100% - 24px);

    margin-top: 15px;

    height: 300px;

  }


  .home-content,
  .page-container {

    width: calc(100% - 24px);

  }


  .feature-text {

    font-size: 17px;

  }


  .feature-button {

    grid-template-columns:
      52px 1fr 30px;

    padding: 0 15px;

  }


  .overview-box {

    padding: 28px;

  }


  .page-header h1 {

    font-size: 30px;

    letter-spacing: 4px;

  }


  .guide-grid {

    grid-template-columns: 1fr;

  }


  .stats-grid {

    grid-template-columns: 1fr;

  }


  .feedback-page-card {

    padding: 25px;

  }


  .digit-selector {

    grid-template-columns:
      repeat(5, 1fr);

  }

}

</style>