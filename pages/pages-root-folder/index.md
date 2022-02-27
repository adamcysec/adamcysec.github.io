---
#
# Use the widgets beneath and the content will be
# inserted automagically in the webpage. To make
# this work, you have to use › layout: frontpage
#
layout: frontpage
header:
  image_fullwidth: header_cyber.jpg
widget1:
  title: "My Blog"
  url: 'http://adamcysec.github.io/blog/'
  image: /widget/widget_cyber_blog.jpg
  text: 'A blog centered around cyber security<br/>topics.<br/>A way for me to share my cyber adventures.'
widget2:
  title: "About Me"
  url: 'http://adamcysec.github.io/info/'
  image: /widget/widget_about_me_cyber_logo.jpg
  text: 'Learn more about how i got started in cyber security.'
widget3:
  title: "My Resume"
  url: 'https://www.linkedin.com/in/adamponce/'
  image: /widget/widget_resume_cyber.jpg
  text: 'View my job experience, certifications, and achievements on LinkedIn!'
#
# Use the call for action to show a button on the frontpage
#
# To make internal links, just use a permalink like this
# url: /getting-started/
#
# To style the button in different colors, use no value
# to use the main color or success, alert or secondary.
# To change colors see sass/_01_settings_colors.scss
#
#callforaction:
#  url: https://tinyletter.com/feeling-responsive
#  text: Inform me about new updates and features ›
#  style: alert
permalink: /index.html
#
# This is a nasty hack to make the navigation highlight
# this page as active in the topbar navigation
#
homepage: true
---

