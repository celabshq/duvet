public/script.js public/style.css: node_modules $(wildcard src/*.js)
	@rm -rf build
	@npm run build
	@awk 1 build/static/js/*.js > public/script.js
	@sed 's|/\*# sourceMappingURL=.*\*/||g' build/static/css/*.css > public/style.css

node_modules:
	@npm install

dev: node_modules src/result.test.json
	@npm start

.PHONY: dev
