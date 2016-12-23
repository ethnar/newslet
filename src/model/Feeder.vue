<script>
import Rx from 'rxjs/Rx';
import $ from 'jquery';

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

  getNewsStream () {
    if (!this.stream) {
      this.stream = new Rx.ReplaySubject(1);

      window.fetch('http://cors-proxy.htmldriven.com/?url=' + this.rssUrl)
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
            return {
              icon: icon,
              title: $(entry).find('title').text(),
              href: $(entry).find('link').attr('href'),
              timestamp: new Date($(entry).find('published').text()),
              imageUrl: imageUrl,
              content: $('<div/>').html(contentText).text().trim().replace(/\n[\s\t]+\n/g, '\n').replace(/\n\s+/g, '\n')
            }
          });
        console.log(entries);
        this.stream.next(entries);
      });
    }
    return this.stream;
  }
}

Feeder.options = {}; // workaround?
export default Feeder;
</script>
