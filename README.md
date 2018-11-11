### Gush
---
https://github.com/chaps-io/gush

```
gem 'gush', '~> 1.0.0'
bundle exec sidekiq -q gush

bundle exec gush show <workflow_id>
bundle exec gush list
bunlde exec gush viz <NameOfTheWorkflow>
```

```ruby
require_relative './config/environment.rb'
config.autoload_paths += ["#{Rails.root}/app/jogs", "#{Rails.root}/app/workflows"]

require_relative 'lib/workflows/example_workflow.rb'
require_relative 'lib/jobs/some_job.rb'
require_relative 'lib/jobs/some_other_job.rb'

# app/workflows/sample_workflow.rb
class SampleWorkflow < Gush::Workflow
  def configure(url_to_fetch_from)
    run FetchJob1, params: { url: url_to_fetch_from }
    run FetchJob2, params: { some_flag: true, url: 'http://url.com' }
    run PersistJob1, after: FetchJob1
    run PersistJob2, after: FetchJob2
    run Normalize,
        after: [PersistJob1, PersistJob2],
        before: Index
    run Index
  end
end

class SimpleWorkflow < Gush::Workflow
  def configure
    run DownloadJob
  end
end

class SimpleWorkflow < Gush::Workflow
  def configure
    run DownloadJob
    run SaveJob, after: DownloadJob
  end
end

class SimpleWorkflow < Gush::Workflow
  def configure
    run FirstDownloadJob
    run SecondDownloadJob
    run SaveJob, after: [FirstDownloadJob, SecondDownloadJob]
  end
end

class SimpleWorkflow < Gush::Workflow
  def configure
    run FirstDownloadJob, before: SaveJob
    run SecondDownloadJob, before: SaveJob
    run SaveJob
  end
end

class PublishBookWorkflow < Gush::Workflow
  def configure(url, isbn)
    run FetchBook, params: { url: url }
    run PublisheBook, params: { book_isbn: isbn }, after: FetchBook
  end
end

PublishBookWorkflow.create("http://url.com/book.pdf", "987-0470081204")

class FetchBook < Gush::Job
  def perform
  end
end

run FetchBook, params: {url: "http://url.com/book.pdf"}

class FetchBook < Gush::Job
  def perform
    params #=> {url: "http://url.com/book.pdf"}
  end
end

flow = PublishBookWorkflow.create("http://url.com/book.pdf", "978-0470081204")

flow.start!

flow.reload
flow.status

class DownloadVideo < Gush::Job
  def perform
    downloader = VideoDownloader.fetch("http://youtube.com/?v=sometvideo")
    output(downloader.file_path)
  end
end

class EncodeVideo < Gush::Job
  def perform
    video_path = payloads.first[:output]
  end
end

[
  {
    id: "DownloadVideo-41bfb730-b49f-42ac-a808-156327989294",
    class: "DownloadVideo",
    output: "https://s3.amazonaws.com/somebucket/downloaded-file.mp4"
  }
]

class NotifyWorkflow < Gush::Workflow
  def configure(user_ids)
    notification_jobs = user_ids.map do |usr_id|
      run NotificationJob, params: {user_id: user_id}
    end
    run AdminNotificationJob, after: notification_jobs
  end
end

flow = NotifyWorkflow.create([54, 21, 24, 154, 65])

# config/initializers/gush.rb
Gush.configure do |config|
  config.redis_url = "redis://localhost:6379"
  config.concurrency
  config.ttl = 3600*24*7
end

# config/initializers/gush.rb
GUSH_CLIENT = Gush::Client.new
def find_by_class klass
  GUSH_CLIENT.all_workflows.each do |flow|
    return true if flow.to_hash[:name] == klass && flow.running?
  end
  return false
end


```


```
```
