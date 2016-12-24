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

class Feeder {
  constructor (args) {
    this.name = args.name;
    this.rssUrl = args.rssUrl;
    this.stream = null;
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
        const xml = $(JSON.parse(news).body);
        const icon = xml.find('> icon').text;
        const entries = xml
          .find('entry')
          .toArray()
          .map(entry => {
            let imageUrl;
            let contentText = $(entry).find('content').text();
            try {
              imageUrl = contentText.match(/src="([^"]+)"/)[1];
            } catch (e) {}
            let href = $(entry).find('link').attr('href');
            return {
              id: this.name + '_' + href,
              icon: icon,
              title: $(entry).find('title').text(),
              href: href,
              timestamp: new Date($(entry).find('published').text()).getTime(),
              imageUrl: imageUrl,
              content: $('<div/>').html(contentText).text().trim().replace(/\n[\s\t]+\n/g, '\n').replace(/\n\s+/g, '\n')
            }
          });
        this.stream
          .first()
          .subscribe(oldEntries => {
            this.stream.next(mergeEntries(oldEntries, entries));
          });
      });
  }

  getNewsStream () {
    if (!this.stream) {
      this.stream = new Rx.ReplaySubject(1);
      this.stream.next([]);

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
