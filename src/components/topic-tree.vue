<template>
  <div id="diagram">
    <svg></svg>
    <div class="buttons">
      <i class="material-icons">add</i>
      <i class="material-icons">remove</i>
    </div>
  </div>
</template>

<script>
import * as d3 from 'd3'

export default {
  name: 'topic-tree',
  props: ['data'],
  data: () => ({
    root: [],
    nodes: [],
    tree: null,
    zoom: null,
    center: { x: 0, y: 0 },
    parentRadius: 40,
    childRadius: 20,
    distance: 200,
    distanceRatio: 2,
    between: 50,
    duration: 200,
    counter: 0
  }),
  mounted () {
    this.initialize()
    this.draw()
  },
  methods: {
    initialize () {
      this.center.x = document.getElementById('diagram').clientWidth / 2
      this.center.y = document.getElementById('diagram').clientHeight / 2

      this.tree = d3.tree()
      this.root = d3.hierarchy(this.data, (d) => (d.children))
      this.root.children.forEach(this.collapse)
    },
    collapse (d) {
      if (d.children) {
        d._children = d.children
        d._children.forEach(this.collapse)
        if (d.depth > 0) {
          d.children = null
        }
      }
    },
    draw () {
      const that = this

      const svg = d3.select('svg')
      this.panel = d3.select('svg').append('g')
        .attr('class', 'content')
        .attr('transform', `translate(${this.center.x}, ${this.center.y})`)
        .attr('cursor', 'pointer')

      this.zoom = d3.zoom().scaleExtent([0.5, 3]).on('zoom', function () {
        that.panel.attr('transform', d3.event.transform)
      })
      const transform = d3.zoomIdentity.translate(this.center.x, this.center.y).scale(1)
      svg.call(that.zoom.transform, transform)
      svg.call(that.zoom)

      this.update(this.root, true)
    },
    update (source, isFirst) {
      const that = this
      const treeData = this.tree(source)
      this.nodes = treeData.descendants()
      // const links = treeData.descendants().slice(1)
      this.nodes.forEach(function (d) {
        if (d.children && d.children.length) {
          let angle = 360 / d.children.length
          if (d.depth < 1) {
            d.children.forEach(function (ch, i) {
              ch.data.angle = 360 - angle * (i + 1)
            })
          }
        }
      })

      const node = this.panel.selectAll('g.node')
        .data(that.nodes, function (d) {
          return d.id || (d.id = ++that.counter)
        })

      // this.drawLink(links)
      isFirst ? this.drawCircle(node) : this.drawMergedCircle(node)
    },
    // draw the circle
    drawCircle (node) {
      const that = this
      const nodeEnter = node.enter().append('g')
        .attr('class', 'circle')

      nodeEnter.append('circle')
        .attr('class', 'circle')
        .attr('r', (d) => ((d.data.isParent) ? that.parentRadius : that.childRadius))
        .style('fill', (d) => (d.data.hasChild ? '#8ee000' : '#f0f'))
        .on('click', function (d) {
          d.data.isParent = true
          return that.nodeClick(d, this)
        })
        .transition()
        .delay((d, i) => (i * that.duration))
        .duration(3 * that.duration)
        .attr('cx', d => {
          d.data.sx = 0
          d.data.cx = d.data.angle === null ? 0 : that.distance * Math.cos((d.data.angle / 180) * Math.PI)
          return d.data.cx
        })
        .attr('cy', d => {
          d.data.sy = 0
          d.data.cy = d.data.angle === null ? 0 : that.distance * Math.sin((d.data.angle / 180) * Math.PI)
          return d.data.cy
        })
    },
    drawMergedCircle (node) {
      const that = this
      const nodeEnter = node.enter().append('g')
        .attr('class', 'circle')

      const nodeUpdate = nodeEnter.merge(node)
      const radio = 360 / nodeUpdate._groups[0].length
      let angle = 0
      let originalPosition = { x: 0, y: 0 }

      nodeUpdate.append('circle')
        .attr('class', 'circle')
        .on('click', function (d) {
          d.data.isParent = true
          return that.nodeClick(d, this)
        })
        .style('fill', (d) => (d.data.hasChild ? '#8ee000' : '#f0f'))
        .attr('r', that.childRadius)
        .attr('cx', function (d, i) {
          if (i < 1) {
            d.data.isParent = true
            angle = d.data.angle
            d.data.sx = d.data.cx
            originalPosition.x = d.data.cx
          } else {
            d.data.cx = (that.distance + that.between) * Math.cos((angle / 180) * Math.PI) + originalPosition.x
            d.data.sx = (that.distance + that.between) * Math.cos((angle / 180) * Math.PI) + originalPosition.x
          }

          return d.data.sx
        })
        .attr('cy', function (d, i) {
          if (i < 1) {
            angle = d.data.angle
            d.data.sy = d.data.cy
            originalPosition.y = d.data.cy
          } else {
            d.data.cy = (that.distance + that.between) * Math.sin((angle / 180) * Math.PI) + originalPosition.y
            d.data.sy = (that.distance + that.between) * Math.sin((angle / 180) * Math.PI) + originalPosition.y
          }
          return d.data.sy
        })
        .transition()
        .delay((d, i) => (i * that.duration))
        .duration(6 * that.duration)
        .attr('cx', function (d, i) {
          if (i < 1) {
            angle = d.data.angle + 180
            d.data.cx = (that.distance + that.between) * Math.cos((d.data.angle / 180) * Math.PI) + d.data.sx
          } else {
            angle = angle - radio
            d.data.cx = that.distance * Math.cos((angle / 180) * Math.PI) + d.data.sx
          }
          return d.data.cx
        })
        .attr('cy', function (d, i) {
          if (i < 1) {
            angle = d.data.angle + 180
            d.data.cy = (that.distance + that.between) * Math.sin((d.data.angle / 180) * Math.PI) + d.data.sy
          } else {
            angle = angle - radio
            d.data.angle = angle
            d.data.cy = that.distance * Math.sin((d.data.angle / 180) * Math.PI) + d.data.sy
          }

          return d.data.cy
        })
        .attr('r', (d) => ((d.data.isParent) ? that.parentRadius : that.childRadius))
    },
    nodeClick (d, ele) {
      let that = this

      if (d.children) {
        d._children = d.children
        d.children = null
      } else {
        d.children = d._children
        d._children = null
      }

      d3.select(ele).remove()
      this.update(d, false)
      d.data.isParent = false

      // Move the diagram
      d3.select('.content')
        .transition()
        .duration(3 * that.duration)
        .attr('transform', function () {
          that.center.x = that.center.x - (2 * that.distance + that.between) * Math.cos((d.data.angle / 180) * Math.PI)
          that.center.y = that.center.y - (2 * that.distance + that.between) * Math.sin((d.data.angle / 180) * Math.PI)

          return `translate(${that.center.x}, ${that.center.y})`
        })

      const transform = d3.zoomIdentity.translate(that.center.x, that.center.y).scale(1)
      d3.select('svg').call(that.zoom.transform, transform)
    },
    drawLink (links) {
      const that = this

      this.panel.selectAll('line')
        .data(links, function (d) {
          return d
        })
        .enter().append('line')
        .style('fill', (d) => (d.data.hasChild ? '#8ee000' : '#f0f'))
        .transition()
        .delay((d, i) => (i * that.duration))
        .duration(5 * that.duration)
        .attr('class', 'line')
        .attr('stroke', '#e0ddd5')
        .attr('stroke-width', '2px')
        .attr('x1', 0)
        .attr('y1', 0)
        .attr('x2', function (d) {
          const angle = (d.data.angle / 180) * Math.PI
          return that.distance * Math.sin(angle)
        })
        .attr('y2', function (d) {
          const angle = (d.data.angle / 180) * Math.PI
          return that.distance * Math.cos(angle)
        })
    }
  }
}
</script>

<style lang="scss" scoped>
#diagram {
  width: 100%;
  height: 100%;
  position: fixed;
}

svg {
  position: relative;
  width: 100%;
  height: 100%;
  background: #f9f9f5;
}

.buttons {
  position: absolute;
  top: 0;
  right: 0;

  i {
    background: white;
    box-shadow: 0 0 10px gray;
    cursor: pointer;
    margin: 10px;
  }
}
</style>
