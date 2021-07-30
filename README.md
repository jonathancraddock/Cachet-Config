# Some Notes on Cachet Config

See: [Cachet](https://github.com/cachethq/Cachet)

## Installing on Ubuntu 20 LTS

*coming soon!* ;-)

-----

**CSS Backup**
```css
@font-face { font-family: GDS-Transport-Light; src: url(/fonts/gds-transport-light.woff2); }
@font-face { font-family: GDS-Transport-Bold; src: url(/fonts/gds-transport-bold.woff2); }

/* override default Cachet font with GDS-Transport */
body.status-page {
    font-family: GDS-Transport-Light, -apple-system,BlinkMacSystemFont,Open Sans,Helvetica Neue,Helvetica,Arial,sans-serif;
    font-size: 1.66em;
}

/* styles the gov.uk black header bar */
.header-banner {
    background-color: black;
    color: #fdfdfd;
    padding-bottom: 10px;
    margin-top: -40px;
    margin-bottom: 20px;
    border-bottom-color: #a657a6;
    border-bottom-width: 5px;
    border-bottom-style: solid;
    height: 52px;
    padding-top: 7px;
}

/* crown logo on black header bar */
.header-logo {
  height: 28px;
  width: auto;
  margin-bottom: 17px;
}

/* white bold text on black header bar, currently "GOV.UK" */
.header-link {
  font-weight: bolder;
  font-size: x-large;
  line-height: 0;
  padding-left: 6px;
  font-family: GDS-Transport-Bold;
}

/* this is the "about" box */
body.status-page .about-app {
  margin-bottom: 20px;
  background-color: #1d70b8;
  padding: 10px 20px;
  color: white;
}

/* targets the "about" markdown, which I've put on top of gov.uk blue */
div.about-app p, strong {
  color: white !important;
  padding-top: 8px;
}

/* targets links inside of the "about" blue box */
div.about-app p a {
  color: #c4bbff;
  text-decoration: none;
  font-weight: 700;
  border-bottom: 2px solid #c4bbff;
}

/* reduces spacing and bolds H1 title text */
.h1, h1 {
  font-family: GDS-Transport-Bold;
  display: inline-block;
  margin-top: 8px;
  line-height: 0.8;
}

/* hide the "history" title */
div.section-timeline h1 {
  display: none;
}

/* hide the "history" title */
  div.section-timeline h4 {
  margin-bottom: 10px;
}

/* tighten spacing of date relative to incident history panels */
div.section-timeline h4 {
  margin-bottom: 6px !important;
  margin-left: 6px;
  font-size: medium;
}

/* targets the "About This Page" default title, and hides it */
div.about-app .h2, h2 {
  font-size: 4rem;
  line-height: 0.7;
  background-color: #1d70b8;
  font-weight: bold;
  display: none;
}

/* Tightens up the spacing at the beginning of the "previous incidents" section */
body.status-page .timeline .content-wrapper {
    margin-top: 0px;
    margin-bottom: 32px;
}

/* Slightly tightens up the "previous incident" text boxes */
body.status-page h1, body.status-page h2, body.status-page h3, body.status-page h4, body.status-page h5 {
    margin-bottom: 18px;
}

/* Prevents group title being overridden by white text heading defaults */
.list-group-item strong {
    color: #333 !important;
}
```
