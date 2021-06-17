# pixim-box2d

"pixim-box2d" is a plugin for using "[box2dweb](https://github.com/hecht-software/box2dweb)" with "[Pixim.js](https://github.com/tawaship/Pixim.js)".

[![Build Status](https://travis-ci.com/tawaship/Pixim-box2d.svg?branch=master)](https://travis-ci.com/tawaship/Pixim-box2d)
[![MIT License](http://img.shields.io/badge/license-MIT-blue.svg?style=flat)](LICENSE)

---

## Supported version

- box2dweb 2.1.0-b
- pixi.js 5.3.x
- @tawaship/pixim.js 1.11.x

I have not confirmed the operation on other versions.

## Setup

### NPM

```sh
npm install --save pixi.js @tawaship/pixim.js @tawaship/pixim-box2d
```

<br />

```javascript
import * as PIXI from 'pixi.js';
import * as Pixim from '@tawaship/pixim.js';
import * as PiximBox2d from '@tawaship/pixim-box2d';
```

### Browser

```sh
git clone https://github.com/tawaship/Pixim-box2d
```

<br />

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/pixi.js/5.3.3/pixi.min.js"></script>
<script src="/path/to/lib/Box2d.min.js"></script>
<script src="/path/to/lib/Pixim.min.js"></script>
<script src="/path/to/dist/Pixim-box2d.min.js"></script>
```

## Usage

For browsers, this module is stored in the namespace "Pixim.box2d".

```javascript
const app = new Pixim.Application({
	width: 450,
	height: 800
});

class Root extends Pixim.Container {
	constructor() {
		super();
		
		this.addChild(new PIXI.Graphics()).beginFill(0x00FF00).drawRect(0, 0, 450, 800 / 2)
		this.interactive = true;
		this.cursor = 'pointer'
		
		// create World
		const world = this.addChild(new Pixim.box2d.WorldContainer({
			listenPreSolve: false,
			listenPostSolve: false,
			listenBeginContact: false,
			listenEndContact: true,
			ticker: app.ticker
		}));
		
		// create component
		const a = world.addBox2d(Pixim.box2d.Circle.from(new PIXI.Graphics().beginFill(0xFF0000).drawCircle(100, 0, 20), {
			density: 0.1,
			restitution: 0.1,
			categoryBits: 2
		}));
		a.setX(120);
		
		// create component when click
		this.on("pointerdown", (e) => {
			const r = Math.random() > 0.5;
			const w = 100
			const h = 150
			
			const a2 = world.addBox2d(new Pixim.box2d.Rectangle(w, h, {
				density: 0.1,
				restitution: 0.1,
				friction: 0.9,
				categoryBits: 1
			}));
			
			a2.addChild(new PIXI.Graphics().beginFill(0xFF0000).drawRect(0, 0, w, h));
			
			if (!r) {
				a2.setX(e.data.global.x - a2.width / 2);
				a2.setY(e.data.global.y - a2.height / 2);
			} else {
				a2.setRotation(90 * Math.PI / 180);
				a2.setX(e.data.global.x + a2.height / 2);
				a2.setY(e.data.global.y - a2.width / 2);
			}
		})
		
		// create static component
		const baseHeight = 100;
		const base = world.addBox2d(Pixim.box2d.Rectangle.from(new PIXI.Graphics().beginFill(0x0000FF).drawRect(0, 0, 450 / 2, baseHeight), { isStatic: true }));
		base.setX(450 / 2 - base.width / 2);
		base.setY(800 - baseHeight - 100);
		
		// create sensor component
		const dead = world.addBox2d(new Pixim.box2d.Edge(new Pixim.box2d.Common.Math.b2Vec2(450 + 2000, 0), { isStatic: true, isSensor: true, maskBits: 1 }));
		dead.setX(-1000);
		dead.setY(800 - 10);
		dead.on('EndContact', (opponent) => {
			world.removeBox2d(opponent);
			console.log("gameover");
		});
	}
}

const Content = Pixim.Content.create();
Content.defineLibraries({
	root: Root
})
const content = new Content();

app.attachAsync(content)
.then(() => {
	app.play();
})
```