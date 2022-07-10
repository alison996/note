### 前置条件
```json
{
  "dependencies": {
    "next": "12.2.0",
    "react": "18.2.0",
    "react-dom": "18.2.0"
  },
  "devDependencies": {
    "@types/node": "^18.0.3",
    "@types/react": "^18.0.15",
    "eslint": "8.19.0",
    "eslint-config-next": "12.2.0",
    "typescript": "^4.7.4"
  }
}
```

### 创建nextjs项目
```
npx create-next-app
```
### 使用`typescript`
```
npm install --save-dev @types/react @types/node
npm run dev
```
<font color="red">**问题：**</font>
1. `tsconfig.json`文件
	报错：Option '--resolveJsonModule' cannot be specified without 'node' module resolution strategy.
	解决：添加`compilerOptions.moduleResolution = 'node'`配置

### nextjs路由
nextjs根据文件路径进行导航。
![image-20220710095647260](https://raw.githubusercontent.com/alison996/cloud/master/img/image-20220710095647260.png)

#### 命令式导航
```tsx
import { Link } from 'next/link';

function Demo() {
	return (
		<ul>
              <li>
                {/* 第一种导航方式 */}
				<Link href='/clients/alison/1996'>
					Alison1996
				</Link>
				{/* 第二种导航方式 */}
				<Link href={
					{
						path: '/clients/[id]/[clientprojectid]',
						query: {
							id: 'alison',
							clientprojectid: '1996'
						}
					}
				}>
					Alison1996
				</Link>
			</li>
		</ul>
	);
}

export default Demo;

```

#### 编程式导航 & 获取路由参数
```tsx
import { useRouter } from 'next/router';

function Demo() {
	const router = useRouter();
	// 获取路由参数 router.query
	console.log('%c%o', '获取路由参数：', router.query);
	
	function loadAlison1996Handler() {
		// 编程式导航 router.push | router.replace
		// 第一种方式
		router.push('/clients/alison/1996');
		// 第二种方式
		router.push({
			path: '/clients/[id]/[clientprojectid]',
			query: {
				id: 'alison',
				clientprojectid: '1996'
			}
		});
	}

	return (
		<div>
			<button onClick={loadAlison1996Handler}>Load Alison1996</button>
		</div>
	);
}

export default Demo;

```