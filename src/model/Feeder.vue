<script>
import Rx from 'rxjs/Rx';
import $ from 'jquery';

function mergeEntries (a, b) {
  let aIds = {};
  a.forEach((news, idx) => {
    aIds[news.id] = idx;
  });
  let results = a.slice();
  b.forEach(news => {
    if (aIds[news.id] !== undefined) {
      results[aIds[news.id]] = news;
    } else {
      results.push(news);
    }
  });
  return results;
}

function parseRss (xml) {
  return xml
    .find('item')
    .toArray()
    .map(entry => {
      entry = $(entry);
      let imageUrl;
      let contentText = entry.find('description').text();
      try {
        imageUrl = contentText.match(/src="([^"]+)"/)[1];
        imageUrl = $('<div/>').html(imageUrl).text();
      } catch (e) {}
      let href = entry.find('link').text();
      return {
        id: href,
        title: entry.find('title').text(),
        href: href,
        timestamp: new Date(entry.find('pubDate').text()).getTime(),
        imageUrl: imageUrl,
        content: $('<div/>').html(contentText).text().trim().replace(/\n[\s\t]+\n/g, '\n').replace(/\n\s+/g, '\n')
      }
    });
}

function parseAtom (xml) {
  // const icon = xml.find('> icon').text;
  return xml
    .find('entry')
    .toArray()
    .map(entry => {
      entry = $(entry);
      let imageUrl;
      let contentText = entry.find('content').text();
      try {
        imageUrl = contentText.match(/src="([^"]+)"/)[1];
      } catch (e) {}
      let href = entry.find('link').attr('href');
      return {
        id: href,
        // icon: icon,
        title: entry.find('title').text(),
        href: href,
        timestamp: new Date(entry.find('published').text()).getTime(),
        imageUrl: imageUrl,
        content: $('<div/>').html(contentText).text().trim().replace(/\n[\s\t]+\n/g, '\n').replace(/\n\s+/g, '\n')
      }
    });
}

function parseFeed (xmlResponse) {
  const xml = $($.parseXML(xmlResponse));
  switch (true) {
    case !!xml.find('rss').length:
      return parseRss(xml);
    default:
      return parseAtom(xml);
  }
}

class Feeder {
  constructor (args) {
    this.name = args.name;
    this.rssUrl = args.rssUrl;
    this.stream = null;
    this.firstLoad = true;
  }

  toJSON () {
    return {
      name: this.name,
      rssUrl: this.rssUrl
    }
  }

  fetch () {
    if (!this.stream) {
      return;
    }
    return window.fetch('http://cors-proxy.htmldriven.com/?url=' + this.rssUrl)
      .then(response => {
        return response.text()
      })
      .then(news => {
        const xmlResponse = JSON.parse(news).body;
        const entries = parseFeed(xmlResponse);
        if (this.firstLoad) {
          this.stream.next(mergeEntries([], entries));
          this.firstLoad = false;
        } else {
          this.stream
            .first()
            .subscribe(oldEntries => {
              this.stream.next(mergeEntries(oldEntries, entries));
            });
        }
      });
  }

  getNewsStream () {
    if (!this.stream) {
      this.stream = new Rx.ReplaySubject(1);
      this.fetch();
      this.interval = setInterval(this.fetch.bind(this), 5 * 60 * 1000);

      this.stream.finally(() => {
        clearInterval(this.interval);
      });
    }
    return this.stream;
  }
}

Feeder.options = {}; // workaround?
export default Feeder;
</script>
