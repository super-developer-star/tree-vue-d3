<template>
  <div id="app">
    <topic-tree :data="treeData"></topic-tree>
  </div>
</template>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

body {
  margin: 0;
}
</style>
<script>
import TopicTree from './components/topic-tree'

export default {
  components: { TopicTree },
  data: () => ({
    treeData: {
      name: 'Science',
      angle: null,
      cx: 0,
      cy: 0,
      isParent: true,
      hasChild: true,
      children: [
        {
          name: 'Physics',
          angle: null,
          cx: 0,
          cy: 0,
          isParent: false,
          hasChild: true,
          children: [
            {
              name: 'Linear Motion 1',
              angle: null,
              cx: 0,
              cy: 0,
              isParent: false,
              hasChild: true,
              children: [
                {
                  name: 'motion',
                  angle: null,
                  cx: 0,
                  cy: 0,
                  isParent: false,
                  hasChild: true,
                  children: [
                    {
                      name: 'color1',
                      angle: null,
                      cx: 0,
                      cy: 0,
                      isParent: false,
                      hasChild: false
                    }
                  ]
                },
                {
                  name: 'SUB-CATEGORY',
                  angle: null,
                  cx: 0,
                  cy: 0,
                  isParent: false,
                  hasChild: true,
                  children: [
                    {
                      name: 'color',
                      angle: null,
                      cx: 0,
                      cy: 0,
                      isParent: false,
                      hasChild: false
                    }
                  ]
                }]
            },
            {
              name: 'Linear motion 2',
              angle: null,
              cx: 0,
              cy: 0,
              isParent: false,
              hasChild: false
            },
            {
              name: 'Linear motion 3',
              angle: null,
              cx: 0,
              cy: 0,
              isParent: false,
              hasChild: false
            }
          ]
        },
        {
          name: 'Chemistry',
          angle: null,
          cx: 0,
          cy: 0,
          isParent: false,
          hasChild: false
        },
        {
          name: 'Physical Science',
          angle: null,
          cx: 0,
          cy: 0,
          isParent: false,
          hasChild: false
        },
        {
          name: 'Biology',
          angle: null,
          cx: 0,
          cy: 0,
          isParent: false,
          hasChild: false
        },
        {
          name: 'Anatomy',
          angle: null,
          cx: 0,
          cy: 0,
          isParent: false,
          hasChild: false
        },
        {
          name: 'Physiology',
          angle: null,
          cx: 0,
          cy: 0,
          isParent: false,
          hasChild: false
        },
        {
          name: 'Zoology',
          angle: null,
          cx: 0,
          cy: 0,
          isParent: false,
          hasChild: false
        },
        {
          name: 'Science',
          angle: null,
          cx: 0,
          cy: 0,
          isParent: false,
          hasChild: false
        },
        {
          name: 'Physics - Intermediate',
          angle: null,
          cx: 0,
          cy: 0,
          isParent: false,
          hasChild: false
        },
        {
          name: 'Biology',
          angle: null,
          cx: 0,
          cy: 0,
          isParent: false,
          hasChild: false
        }
      ]
    }
  })
}
</script>
