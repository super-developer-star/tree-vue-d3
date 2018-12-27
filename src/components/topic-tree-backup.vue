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
    root: null,
    nodes: null,
    center: { x: 0, y: 0 },
    parentRadius: 40,
    childRadius: 20,
    distance: 200,
    distanceRatio: 2,
    between: 50,
    duration: 200,
    counter: 0,
    isCollapseStatus: false
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
      this.circlesTag = this.panel.append('g').attr('class', 'circles')

      this.zoom = d3.zoom().scaleExtent([0.5, 3]).on('zoom', function () {
        that.panel.attr('transform', d3.event.transform)
      })
      const transform = d3.zoomIdentity.translate(this.center.x, this.center.y).scale(1)
      svg.call(that.zoom.transform, transform)
      svg.call(that.zoom)

      this.update(this.root)
    },
    update (source) {
      const that = this
      const treeData = this.tree(this.root)
      this.nodes = treeData.descendants()
      this.nodes.forEach(function (d) {
        if (d.children && d.children.length) {
          let angle = 360 / d.children.length
          if (d.depth < 1) {
            d.children.forEach(function (ch, i) {
              ch.data.angle = angle * (i + 1)
            })
          }
        }
      })

      const node = this.circlesTag.selectAll('circle')
        .data(that.nodes, function (d) {
          return d.id || (d.id = ++that.counter)
        })

      this.drawCircle(node, source)
    },
    // draw the circle
    drawCircle (node, source) {
      const that = this

      const nodeEnter = node.enter()

      const nodeUpdate = nodeEnter.merge(node)
      const radio = 360 / nodeUpdate._groups[0].length

      nodeEnter.append('circle')
        .attr('class', (d) => (d.data.hasChild && d.data.isParent === false ? 'hasChild' : ''))
        .attr('r', (d) => ((d.data.isParent) ? that.parentRadius : that.childRadius))
        .attr('cx', function (d, i) {
          console.log('add: ' + d.data.name + ' : ' + i)
          return source.data.cx
        })
        .attr('cy', function (d) {
          return source.data.cy
        })
        .on('click', function (d) {
          that.nodeClick(d, this)
        })
        .transition()
        .delay((d, i) => (i * that.duration))
        .duration(3 * that.duration)
        .attr('cx', function (d, i) {
          d.data.angle = (i + 1) * radio
          d.data.cx = that.distance * Math.cos((d.data.angle / 180) * Math.PI)
          return d.data.cx
        })
        .attr('cy', function (d, i) {
          d.data.angle = (i + 1) * radio
          d.data.cy = that.distance * Math.sin((d.data.angle / 180) * Math.PI)
          return d.data.cy
        })

      // let originalPosition = { x: 0, y: 0 }

      // nodeUpdate
      //   .transition()
      //   .delay((d, i) => (i * that.duration))
      //   .duration(3 * that.duration)
      //   .attr('cx', function (d, i) {
      //     d.data.angle = angle + i * radio
      //     d.data.cx = that.distance * Math.cos((d.data.angle / 180) * Math.PI)
      //     return d.data.cx
      //   })
      //   .attr('cy', function (d, i) {
      //     d.data.angle = angle + i * radio
      //     d.data.cy = that.distance * Math.sin((d.data.angle / 180) * Math.PI)
      //     return d.data.cy
      //   })

      node.exit()
        .transition()
        .duration(that.duration)
        .attr('cx', function (d, i) {
          console.log('removed: ' + source.data.name + ' : ' + i)
          return source.data.cx
        })
        .remove()
    },
    nodeClick (d, ele) {
      if (d.children) {
        d._children = d.children
        d.children = null
        this.isCollapseStatus = false
      } else {
        d.children = d._children
        d._children = null
        this.isCollapseStatus = true
      }

      this.update(d)
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

<style lang="scss">
circle {
  fill: #8ee000;
  cursor: default;
}

.hasChild {
  stroke: red;
  cursor: pointer;
}

line {
  stroke: #e0ddd5;
  stroke-width: 2px;
}

text {
  text-anchor: middle;
  opacity: 0;
}
</style>
