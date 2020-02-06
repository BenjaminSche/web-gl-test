<template>
  <div ref="visuPixi" class="visu-pixi" />
</template>

<script>
import * as PIXI from 'pixi.js-legacy'
import Bus from '@/event/Bus'
import dagre from 'dagre'
import { Viewport } from 'pixi-viewport'

const SCREEN_WIDTH = 1000
const SCREEN_HEIGHT = 700
const WORLD_WIDTH = SCREEN_WIDTH * 2
const WORLD_HEIGHT = SCREEN_HEIGHT * 2
const NODE_WIDTH = 220
const NODE_HEIGHT = 100
const UNIT_RADIUS = 7
const RESOLUTION = window.devicePixelRatio

export default {
  data() {
    return {
      viewport: undefined,
      steps: [
        {
          id: 1
        },
        {
          id: 2
        }
      ],
      paths: [
        {
          id: '1|2',
          from: 1,
          to: 2
        }
      ],
      units: [
        {
          events: [1, 2]
        }
      ],
      dagreGraph: undefined,
      stepTexture: null
    }
  },
  mounted() {
    Bus.$on('datas-generated', event => this.onDataReceived(event))
    Bus.$on('stream-play', event => this.onPlay(event))

    this.init()
    this.setData(this.steps, this.paths)
    this.draw()

    // // animate
    // app.ticker.add(() => {
    //   circle.x += 0.5
    // })
  },
  methods: {
    onDataReceived(event) {
      this.setData(event.steps, event.links)
      this.draw()
    },
    onPlay() {
      const circleLayer = new PIXI.Container()
      this.viewport.addChild(circleLayer)

      const circle = new PIXI.Sprite(this.unitTexture)

      circle.x = SCREEN_WIDTH / 2
      circle.y = SCREEN_HEIGHT / 2
      circleLayer.addChild(circle)

      // for (let unit of this.units) {
      //   const circle = new PIXI.Sprite(this.circleTexture)
      //   console.log('unit', unit)
      //   // circle.position.set(20, 20)
      //   circle.x = 20
      //   circle.y = 20
      //   circleLayer.addChild(circle)
      // }
      this.app.render()
    },
    setData(steps, links) {
      this.steps = []
      this.paths = []
      this.computeGraph(steps, links)

      this.dagreGraph.nodes().forEach(nodeId => {
        let node = this.dagreGraph.node(nodeId)
        this.steps.push({
          id: nodeId,
          x: node.x - NODE_WIDTH / 2,
          y: node.y - NODE_HEIGHT / 2
        })
      })
      console.log('nodes', this.dagreGraph.nodes().length)

      this.dagreGraph.edges().forEach(edgeId => {
        let edge = this.dagreGraph.edge(edgeId)
        this.paths.push({
          id: edgeId.v + '|' + edgeId.w,
          from: edgeId.v,
          to: edgeId.w,
          points: edge.points
        })
      })
      console.log('edges', this.dagreGraph.edges().length)
    },
    init() {
      this.app = new PIXI.Application({
        width: 1600,
        height: 800,
        resolution: RESOLUTION,
        antialias: true,
        autoStart: false
      })
      this.$refs.visuPixi.appendChild(this.app.view)

      this.viewport = new Viewport({
        screenWidth: SCREEN_WIDTH,
        screenHeight: SCREEN_HEIGHT,
        worldWidth: WORLD_WIDTH,
        worldHeight: WORLD_HEIGHT,
        interaction: this.app.renderer.plugins.interaction
      })

      this.app.stage.addChild(this.viewport)

      this.viewport
        .drag()
        .pinch()
        .wheel()

      //have to set it with autoStart: false on this.app
      this.viewport.on('frame-end', () => {
        if (this.viewport.dirty) {
          this.app.render()
          this.viewport.dirty = false
        }
      })

      const rectGraphics = new PIXI.Graphics()
      rectGraphics.lineStyle(1, PIXI.utils.string2hex('#2e3630'), 1)
      rectGraphics.beginFill(PIXI.utils.string2hex('#cecece'))
      rectGraphics.drawRoundedRect(0, 0, NODE_WIDTH, NODE_HEIGHT, 16)
      this.stepTexture = rectGraphics.generateCanvasTexture(
        PIXI.SCALE_MODES.LINEAR,
        RESOLUTION
      )

      const circleGraphics = new PIXI.Graphics()
      circleGraphics.beginFill(0xff00ff)
      circleGraphics.drawCircle(0, 0, UNIT_RADIUS)
      this.unitTexture = circleGraphics.generateCanvasTexture(
        PIXI.SCALE_MODES.LINEAR,
        RESOLUTION
      )
    },
    draw() {
      this.removeAll()

      const nodesLayer = new PIXI.Container()
      const linksLayer = new PIXI.Container()
      this.viewport.addChild(nodesLayer)
      this.viewport.addChild(linksLayer)

      // const circleGraphics = new PIXI.Graphics()
      // circleGraphics.beginFill(TEXTURE_COLOR)
      // circleGraphics.drawCircle(0, 0, NODE_RADIUS)

      // circleGraphics.x = SCREEN_WIDTH / 2
      // circleGraphics.y = SCREEN_HEIGHT / 2

      for (let step of this.steps) {
        const rectGraphics = new PIXI.Sprite(this.stepTexture)
        rectGraphics.position.set(step.x, step.y)

        const titleText = new PIXI.Text(step.id, {
          fontFamily: 'Arial',
          fontSize: 24,
          fill: 'black',
          align: 'right'
        })
        titleText.anchor.set(0.5, 0.5)
        titleText.position.set(NODE_WIDTH / 2, NODE_HEIGHT / 2)

        rectGraphics.addChild(titleText)
        nodesLayer.addChild(rectGraphics)
      }

      for (let path of this.paths) {
        const curveGraphics = this.createCurveFromPoints(path.points)
        linksLayer.addChild(curveGraphics)
        // for (let point of path.points) {
        //   const circleGraphics = new PIXI.Graphics()
        //   circleGraphics.beginFill(0xff00ff)
        //   circleGraphics.drawCircle(point.x, point.y, 5)
        //   nodesLayer.addChild(circleGraphics)
        // }
      }

      this.app.render()
    },
    createCurveFromPoints(points) {
      const curveGraphics = new PIXI.Graphics()
      curveGraphics.lineStyle(1, 0xffffff)

      // move to the first point
      curveGraphics.moveTo(points[0].x, points[0].y)
      let i
      for (i = 1; i < points.length - 2; i++) {
        var xc = (points[i].x + points[i + 1].x) / 2
        var yc = (points[i].y + points[i + 1].y) / 2
        curveGraphics.quadraticCurveTo(points[i].x, points[i].y, xc, yc)
      }
      // curve through the last two points
      curveGraphics.quadraticCurveTo(
        points[i].x,
        points[i].y,
        points[i + 1].x,
        points[i + 1].y
      )

      return curveGraphics
    },
    computeGraph(nodes, links) {
      this.dagreGraph = new dagre.graphlib.Graph()
        .setGraph({
          rankdir: 'LR',
          marginx: 40,
          marginy: 100,
          ranksep: 100,
          edgesep: 100
        })
        .setDefaultEdgeLabel(function() {
          return {}
        })

      for (let node of nodes) {
        this.dagreGraph.setNode(node.id, {
          width: NODE_WIDTH,
          height: NODE_HEIGHT,
          id: node.id
        })
      }

      for (let link of links) {
        this.dagreGraph.setEdge(link.from, link.to)
      }

      dagre.layout(this.dagreGraph)
    },
    removeAll() {
      while (this.viewport.children[0]) {
        this.viewport.removeChild(this.viewport.children[0])
      }
    }
  }
}
</script>

<style scoped></style>
