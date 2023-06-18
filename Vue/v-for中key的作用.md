```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <!-- 导入Vue脚本 -->
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
</head>

<body>
    <div id="demoDiv">
        <input type="text" v-model="name" />
        <button @click="addCheckBox">添加</button>
        <ul>
            <li v-for="(p, i) in product" :key="p.id">
                <input type="checkbox" /> {{p.name}}
            </li>
        </ul>
    </div>
    <script>
        const demo = {
            data: function () {
                return {
                    product: [
                        { id: 1, name: '开口松子' },
                        { id: 2, name: '成语大词典' }
                    ],
                    name: '',
                    nextId: 3
                }
            },
            methods: {
                addCheckBox() {
                    this.product.unshift({ id: this.nextId, name: this.name })
                    this.name = ''
                    this.nextId++
                }
            }
        }
        const demoDiv = Vue.createApp(demo);
        demoDiv.mount('#demoDiv');
    </script>
</body>

</html>
```

