<script setup>
import { ref } from "vue";
import axios from 'axios';

getList()


const value = ref('');
const list = ref([
]);

async function getList() {

  const res = await axios({
    url: "https://gi3ielbr3v.hzh.sealos.run/get_list",
    method: "GET"
  })
  list.value = res.data.list;
}

async function add() {

  await axios({
    url: "https://gi3ielbr3v.hzh.sealos.run/post_list",
    method: "POST",
    data: {
      text: value.value,
      iscompleted: false
    }
  })
  value.value="";
  getList()
}

async function update(id) {
  await axios({
    url: "https://gi3ielbr3v.hzh.sealos.run/update_todo",
    method: "POST",
    data: {
      id,
    }
  })
  getList()
}

async function remove(id) {
  await axios({
    url:"https://gi3ielbr3v.hzh.sealos.run/remove_todo",
    method:"POST",
    data:{
      id:id,
    }
  })
  getList()
}

function handleKeydown(event) {
  if (event.key === 'Enter') {
    add(); // 如果按下回车键，调用添加函数
  }
}
</script>

<template>
  <div>
    <div class="box1">
      <div class="h1">Todo App</div>
      <div class="todo-form">
        <input v-model="value" @keydown="handleKeydown"
        type="text" class="todo-input" placeholder="add todo">
        <div @click="add()" class="todo-button">add todo</div>
      </div>

      <div v-for="(item, index) in list" :key="index" :class="[item.iscompleted ? 'completed' : 'item']">
        <div>
          <input @click="update(item._id)" v-model="item.iscompleted" type="checkbox">
          <span class="name">{{ item.text }}</span>
        </div>
        <div @click="remove(item._id)" class="del">del</div>
      </div>

    </div>
  </div>

</template>



<style>
.completed {
  text-decoration: line-through;
  opacity: 0.4;
  display: flex;
  border-radius: 20px;
  width: 60%;
  height: 50px;
  box-shadow: rgba(149, 157, 165, 0.2) 0px 8px 20px;
  margin: 20px auto;
  margin-left: 20%;
  align-items: center;
  justify-content: space-between;
  padding-left: 20px;
}

.h1 {
  margin-top: 50px;
  font-size: 30px;
  font-weight: 500;
}

.todo-input {
  border: 1px solid #dfe1e5;
  outline: none;
  width: 60%;
  height: 50px;
  border-radius: 20px 0 0 20px;
  padding-left: 15px;
  margin-left: 10%;
  margin-bottom: 30px;
}

.todo-button {
  width: 80px;
  height: 50px;
  border-radius: 0 20px 20px 0;
  background-image: linear-gradient(to right top, #000000, #151515, #222222, #303030, #3e3e3e);  text-align: center;
  ;
  font-size: 15px;
  color: #dfe1e5;
  line-height: 50px;
  cursor: pointer;
  user-select: none;
}

.todo-form {
  display: flex;
  margin-top: 20px;
}

.item {
  display: flex;
  border-radius: 20px;
  width: 60%;
  height: 50px;
  box-shadow: rgba(149, 157, 165, 0.2) 0px 8px 20px;
  margin: 20px auto;
  margin-left: 20%;
  align-items: center;
  justify-content: space-between;
  padding-left: 20px;
  padding-right: 20px;
}

.del {
  color: red;
  cursor: pointer;
}

.box1 {
  width: 98%;
  height: 500px;
  background-color: white;
  border-radius: 6px;
  text-align: center;
  margin-top: 40px;
  margin-left: 1%;
  padding-top: 1px;
  box-sizing: border-box;
}

body {
  background-image: linear-gradient(to right top, #000000, #151515, #222222, #303030, #3e3e3e);}
</style>
<!-- 四个接口代码 -->
<!-- 
get_list查询数据库
import cloud from '@lafjs/cloud'

export default async function (ctx: FunctionContext) {
  const db = cloud.mongo.db;

  try {
    // 查询 'list' 表中的所有数据
    const data = await db.collection('list').find().toArray();

    // 返回查询结果
    return {
      success: true,
      list: data,
    };
  } catch (error) {
    // 错误处理
    return {
      success: false,
      error: error.message
    };
  }
}


post_list增加数据库元素
import cloud from '@lafjs/cloud';

export default async function (ctx: FunctionContext) {
  const db = cloud.mongo.db; // 获取 MongoDB 实例

  // 假设要添加的数据对象
  const data = {
    text: ctx.body.text, // 从请求上下文中获取数据
    iscompleted: false, // 默认值
  };

  try {
    // 向 list 集合插入数据
    const result = await db.collection('list').insertOne(data);

    // 返回成功的响应
    return {
      code: 200,
      message: '添加成功',
      data: data.text,
    };
  } catch (error) {
    // 处理错误
    return {
      code: 500,
      message: '添加失败',
      error: error.message,
    };
  }
}


remove_todo删除数据库元素
import cloud from '@lafjs/cloud';
import { ObjectId } from 'mongodb';

export default async function (ctx: FunctionContext) {
  const db = cloud.mongo.db; // 获取 MongoDB 实例
  const id = ctx.body.id; // 获取请求体中的 id

  if (!ObjectId.isValid(id)) {
    return {
      success: false,
      message: "无效的ID格式"
    };
  }

  try {
    // 从 'list' 表中删除指定 id 的数据
    const collection = db.collection('list');
    const result = await collection.deleteOne({ _id: new ObjectId(id) });


    if (result.deletedCount > 0) {
      return {
        success: true,
        message: '删除成功'
      };
    } else {
      return {
        success: false,
        message: '未找到指定的ID，删除失败'
      };
    }
  } catch (error) {
    return {
      success: false,
      error: error.message
    };
  }
}


update_todo修改数据库元素属性
import cloud from '@lafjs/cloud';
import { ObjectId } from 'mongodb'; // 确保导入 ObjectId

export default async function (ctx: FunctionContext) {
  const db = cloud.mongo.db; // 获取 MongoDB 实例
  const { id } = ctx.body; // 从请求中获取 id、text 和 iscompleted

  try {
    // 查找文档
    const document = await db.collection('list').findOne({ _id: new ObjectId(id) });

    // 如果文档存在，更新 iscompleted 属性
    if (document) {
      const newCompletedStatus = !document.iscompleted; // 切换 iscompleted 的值
      await db.collection('list').updateOne(
        { _id: new ObjectId(id) },
        { $set: { iscompleted: newCompletedStatus } } // 更新 iscompleted 属性
      );

      return {
        success: true,
        message: '更新成功',
      };
    } else {
      return {
        success: false,
        message: '文档未找到',
      };
    }
  } catch (error) {
    return {
      success: false,
      error: error.message,
    };
  }
}
-->