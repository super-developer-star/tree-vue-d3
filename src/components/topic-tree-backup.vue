<template>
  <div id="diagram">
    <svg></svg>
    <!--<div class="buttons">-->
      <!--<i class="material-icons">add</i>-->
      <!--<i class="material-icons">remove</i>-->
    <!--</div>-->
  </div>
</template>

<script>
import * as d3 from 'd3'

export default {
  name: 'topic-tree',
  props: ['data'],
  data: () => ({
    center: { x: 0, y: 0 },
    parentRadius: 40,
    childRadius: 20,
    distance: 200,
    distanceRatio: 2,
    between: 50,
    duration: 200,
    counter: 0,
    circleColor: '#8ee000',
    lineColor: '#e0ddd5'
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
      this.linksTag = this.panel.append('g').attr('class', 'links')
      this.circlesTag = this.panel.append('g').attr('class', 'circles')
      this.titlesTag = this.panel.append('g').attr('class', 'titles')

      this.zoom = d3.zoom().scaleExtent([0.5, 3]).on('zoom', function () {
        that.panel.attr('transform', d3.event.transform)
      })
      const transform = d3.zoomIdentity.translate(this.center.x, this.center.y).scale(1)
      svg.call(that.zoom.transform, transform)
      svg.call(that.zoom)

      this.update(this.root, true, true)
    },
    update (source, isFirst, isTrans) {
      const that = this
      const treeData = this.tree(source)
      this.nodes = treeData.descendants()
      const links = treeData.descendants()
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

      const link = this.linksTag.selectAll('g')
        .data(links, function (d) {
          return d
        })

      const node = this.circlesTag.selectAll('g')
        .data(that.nodes, function (d) {
          return d.id || (d.id = ++that.counter)
        })

      const title = this.titlesTag.selectAll('g')
        .data(that.nodes, function (d) {
          return d.id || (d.id = ++that.counter)
        })

      isFirst ? this.drawCircle(node) : this.drawMergedCircle(node, isTrans)
      isFirst ? this.drawLink(link) : this.drawMergedLink(link, isTrans)
      isFirst ? this.drawText(title, isTrans) : this.drawMergedText(title, isTrans)
    },
    // draw the circle
    drawCircle (node) {
      const that = this
      const nodeEnter = node.enter()

      nodeEnter.append('circle')
        .attr('id', d => ('circle-' + d.id))
        .attr('class', (d) => (d.data.hasChild && d.data.isParent === false ? 'hasChild' : ''))
        .attr('r', (d) => ((d.data.isParent) ? that.parentRadius : that.childRadius))
        .on('click', function (d) {
          if (d.data.hasChild && d.data.isParent === false) {
            d.data.isParent = true
            return that.nodeClick(d, this)
          }
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
    isClickable (d) {
      let isClickable = true
      d.children.forEach(function (v) {
        if (v.children !== undefined && v.children !== null) {
          isClickable = false
          return false
        }
      })

      return isClickable
    },
    drawMergedCircle (node, isTrans) {
      const that = this
      const nodeEnter = node.enter()

      const nodeUpdate = nodeEnter.merge(node)
      const radio = 360 / nodeUpdate._groups[0].length
      let angle = 0
      let originalPosition = { x: 0, y: 0 }

      const nodeUdateEle = nodeUpdate.append('circle')
        .attr('id', d => ('circle-' + d.id))
        .attr('class', (d) => (d.data.hasChild && d.data.isParent === false ? 'hasChild' : ''))
        .on('click', function (d) {
          if (d.data.isParent) {
            if (that.isClickable(d)) {
              that.nodeCollapse(d, this)
            } else {
              return false
            }
          } else {
            if (d.data.hasChild) {
              d.data.isParent = true
              that.nodeClick(d, this)
            }
          }
        })
        .attr('r', that.childRadius)
        .style('opacity', (d) => (d.isShown ? 1 : 0))
        .attr('cx', function (d, i) {
          if (i < 1) {
            if (!isTrans) {
              d.data.sx = d.parent.data.cx
              d.isShown = false
              return d.data.cx
            } else {
              d.isShown = true
              angle = d.data.angle
              d.data.sx = d.data.cx
              originalPosition.x = d.data.cx
            }
          } else {
            d.data.cx = (that.distance + that.between) * Math.cos((angle / 180) * Math.PI) + originalPosition.x
            d.data.sx = (that.distance + that.between) * Math.cos((angle / 180) * Math.PI) + originalPosition.x
          }
          return d.data.sx
        })
        .attr('cy', function (d, i) {
          if (i < 1) {
            if (!isTrans) {
              d.data.sy = d.parent.data.cy
              return d.data.cy
            } else {
              angle = d.data.angle
              d.data.sy = d.data.cy
              originalPosition.y = d.data.cy
            }
          } else {
            d.data.cy = (that.distance + that.between) * Math.sin((angle / 180) * Math.PI) + originalPosition.y
            d.data.sy = (that.distance + that.between) * Math.sin((angle / 180) * Math.PI) + originalPosition.y
          }
          return d.data.sy
        })

      if (isTrans) {
        nodeUdateEle
          .transition()
          .delay((d, i) => (3 * i * that.duration))
          .duration(6 * that.duration)
          .style('opacity', 1)
          .attr('cx', function (d, i) {
            if (i < 1) {
              angle = d.data.angle + 180
              d.data.cx = (that.distance + that.between) * Math.cos((d.data.angle / 180) * Math.PI) + d.data.sx
            } else {
              angle = angle + radio
              d.data.cx = that.distance * Math.cos((angle / 180) * Math.PI) + d.data.sx
            }
            return d.data.cx
          })
          .attr('cy', function (d, i) {
            if (i < 1) {
              angle = d.data.angle + 180
              d.data.cy = (that.distance + that.between) * Math.sin((d.data.angle / 180) * Math.PI) + d.data.sy
            } else {
              angle = angle + radio
              d.data.angle = angle
              d.data.cy = that.distance * Math.sin((d.data.angle / 180) * Math.PI) + d.data.sy
            }

            return d.data.cy
          })
          .attr('r', (d) => ((d.data.isParent) ? that.parentRadius : that.childRadius))
      }
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
      d3.select('#line-' + d.id).remove()
      d3.select('#text-' + d.id).remove()
      d.isShown = true
      this.update(d, false, true)
      d3.select('#circle-' + d.id).attr('class', 'isClickable')
      d3.select('#circle-' + d.parent.id).attr('class', '')
      // Move the diagram
      d3.select('.content')
        .transition()
        .duration(3 * that.duration)
        .attr('transform', function () {
          that.center.x = that.center.x - (2 * that.distance + that.between) * Math.cos((d.data.angle / 180) * Math.PI)
          that.center.y = that.center.y - (2 * that.distance + that.between) * Math.sin((d.data.angle / 180) * Math.PI)

          return `translate(${that.center.x}, ${that.center.y})`
        })
        .on('end', () => {
          const transform = d3.zoomIdentity.translate(that.center.x, that.center.y).scale(1)
          d3.select('svg').call(that.zoom.transform, transform)
        })
    },
    nodeCollapse (d, ele) {
      const that = this
      this.drawRemove(d)
      d3.select('.content')
        .transition()
        .delay(3 * d.children.length * that.duration)
        .duration(6 * that.duration)
        .attr('transform', function () {
          that.center.x = that.center.x + (2 * that.distance + that.between) * Math.cos((d.data.angle / 180) * Math.PI)
          that.center.y = that.center.y + (2 * that.distance + that.between) * Math.sin((d.data.angle / 180) * Math.PI)

          return `translate(${that.center.x}, ${that.center.y})`
        })
        .on('end', () => {
          const transform = d3.zoomIdentity.translate(that.center.x, that.center.y).scale(1)
          d3.select('svg').call(that.zoom.transform, transform)
        })
    },
    drawRemove (node) {
      const that = this
      const treeData = this.tree(node)
      this.nodes = treeData.descendants()

      let ind = 0
      that.nodes.forEach((d, i) => {
        ind = i > 0 ? i - 1 : that.nodes.length

        d.data.isParent = false
        d3.select('#circle-' + d.id)
          .transition()
          .delay(3 * ind * that.duration)
          .duration(6 * that.duration)
          .style('opacity', () => i === 0 ? 1 : 0)
          .attr('r', () => ((d.data.isParent) ? that.parentRadius : that.childRadius))
          .attr('cx', d => d.data.sx)
          .attr('cy', d => d.data.sy)
          .on('end', () => {
            d3.select('#circle-' + d.id).remove()
            if (i === 0) {
              d._children = d.children
              d.children = null
              d.isShown = true
              d.data.cx = d.data.sx
              d.data.cy = d.data.sy
              that.update(d, false, false)
              if (d.parent.parent !== null) {
                d3.select('#circle-' + d.parent.id).attr('class', 'isClickable')
              }
            }
          })

        d3.select('#line-' + d.id)
          .transition()
          .delay(3 * ind * that.duration)
          .duration(6 * that.duration)
          .attr('x1', (d, i) => (i === 0 ? d.parent.data.cx : d.data.sx))
          .attr('y1', (d, i) => (i === 0 ? d.parent.data.cy : d.data.sy))
          .attr('x2', (d) => (d.data.sx))
          .attr('y2', (d) => (d.data.sy))
          .on('end', () => {
            d3.select('#line-' + d.id).remove()
          })

        d3.select('#text-' + d.id)
          .transition()
          .delay(3 * ind * that.duration)
          .duration(6 * that.duration)
          .style('opacity', () => i === 0 ? 1 : 0)
          .attr('transform', function (d) {
            const delta = (d.data.isParent ? that.parentRadius : that.childRadius) + 20
            const x = d.data.sx
            const y = d.data.sy > 0
              ? d.data.sy + delta : d.data.sy - delta
            return `translate(${x},${y})`
          })
          .on('end', () => {
            d3.select('#text-' + d.id).remove()
          })
      })
    },
    drawLink (link) {
      const that = this
      const lineEnter = link.enter()
        .append('line')
        .attr('id', d => ('line-' + d.id))

      lineEnter.transition()
        .delay((d, i) => (i * that.duration))
        .duration(3 * that.duration)
        .attr('x1', 0)
        .attr('y1', 0)
        .attr('x2', function (d) {
          if (d.data.angle === null) return 0
          const angle = (d.data.angle / 180) * Math.PI
          return that.distance * Math.cos(angle)
        })
        .attr('y2', function (d) {
          if (d.data.angle === null) return 0
          const angle = (d.data.angle / 180) * Math.PI
          return that.distance * Math.sin(angle)
        })
    },
    drawMergedLink (link, isTrans) {
      const that = this
      const lineEnter = link.enter().append('line').attr('id', d => ('line-' + d.id))
      const lineUpdate = lineEnter.merge(link)

      lineUpdate
        .attr('x1', (d, i) => (isTrans ? (i === 0 ? d.parent.data.cx : d.data.sx) : d.data.cx))
        .attr('y1', (d, i) => (isTrans ? (i === 0 ? d.parent.data.cy : d.data.sy) : d.data.cy))
        .attr('x2', (d) => (d.data.sx))
        .attr('y2', (d) => (d.data.sy))

      if (isTrans) {
        lineUpdate.transition()
          .delay((d, i) => (3 * i * that.duration))
          .duration(6 * that.duration)
          .attr('x1', (d, i) => (i === 0 ? d.parent.data.cx : d.data.sx))
          .attr('y1', (d, i) => (i === 0 ? d.parent.data.cy : d.data.sy))
          .attr('x2', (d) => (d.data.cx))
          .attr('y2', (d) => (d.data.cy))
      }
    },
    drawText (text) {
      const that = this
      const textEnter = text.enter().append('text')
        .attr('id', d => ('text-' + d.id))
        .html(d => d.data.name)

      textEnter
        .transition()
        .delay((d, i) => (i * that.duration))
        .duration(3 * that.duration)
        .style('opacity', '1')
        .attr('transform', function (d) {
          if (d.data.angle === null) {
            return `translate(0, 55)`
          } else {
            const angle = (d.data.angle / 180) * Math.PI
            const delta = (d.data.isParent ? that.parentRadius : that.childRadius) + 20
            const x = that.distance * Math.cos(angle)
            const y = that.distance * Math.sin(angle) > 0
              ? that.distance * Math.sin(angle) + delta : that.distance * Math.sin(angle) - delta
            return `translate(${x},${y})`
          }
        })
    },
    drawMergedText (text, isTrans) {
      const that = this
      const textEnter = text.enter().append('text')
        .attr('id', d => ('text-' + d.id))
        .html(d => d.data.name)
      const textUpdate = textEnter.merge(text)

      textUpdate
        .style('opacity', (d) => (d.isShown || !isTrans ? 1 : 0))
        .attr('transform', function (d) {
          const delta = that.childRadius + 20
          const x = isTrans ? d.data.sx : d.data.cx
          const y = isTrans ? (d.data.sy > 0 ? d.data.sy + delta : d.data.sy - delta)
            : (d.data.cy > 0 ? d.data.cy + delta : d.data.cy - delta)
          return `translate(${x},${y})`
        })

      if (isTrans) {
        textUpdate.transition()
          .delay((d, i) => (3 * i * that.duration))
          .duration(6 * that.duration)
          .style('opacity', 1)
          .attr('transform', function (d) {
            const delta = (d.data.isParent ? that.parentRadius : that.childRadius) + 20
            const x = d.data.cx
            const y = d.data.cy > 0
              ? d.data.cy + delta : d.data.cy - delta
            return `translate(${x},${y})`
          })
      }
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

.isClickable {
  cursor: pointer;
  fill: blue
}
</style>
