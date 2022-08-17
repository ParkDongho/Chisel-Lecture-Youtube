main:
	make drawio
newDiagram:
	@read -p "Enter File Name:" file_name; \
	cp ./blank.drawio ./diagram/$$file_name.drawio \
	&& open ./diagram/$$file_name.drawio
drawio:
	/Applications/draw.io.app/Contents/MacOS/draw.io -x -f svg -o image/ diagram/
preview:
	marp ./slide.md -p -o ./slide.html
pdf:
	marp ./slide.md --pdf --allow-local-files -o ./slide.pdf
ppt:
	marp ./introduction_to_digital_system_Design.md -p --pptx --allow-local-files -o ./introduction_to_digital_system_Design.pptx

