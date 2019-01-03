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
      hasChild: true,
      children: [
        {
          name: 'Physics',
          hasChild: true,
          children: [
            {
              name: 'Linear Motion 1',
              hasChild: true,
              children: [
                {
                  name: 'motion',
                  hasChild: true,
                  children: [{ name: 'color1', hasChild: false }]
                },
                {
                  name: 'SUB-CATEGORY',
                  hasChild: true,
                  children: [{ name: 'color', hasChild: false }]
                }]
            },
            { name: 'Linear motion 2', hasChild: false },
            { name: 'Linear motion 3', hasChild: false }
          ]
        },
        { name: 'Chemistry', hasChild: false },
        { name: 'Physical Science', hasChild: false },
        { name: 'Biology', hasChild: false },
        { name: 'Anatomy', hasChild: false },
        { name: 'Physiology', hasChild: false },
        { name: 'Zoology', hasChild: false },
        { name: 'Science', hasChild: false },
        { name: 'Physics - Intermediate', hasChild: false },
        { name: 'Biology', hasChild: false }
      ]
    }
  })
}
</script>
