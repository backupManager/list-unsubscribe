require 'mail'
require 'open-uri'
require 'uri'

Mail.defaults do
  retriever_method :imap, {
    :address    => ENV['IMAP_ADDRESS'],
    :user_name  => ENV['IMAP_USERNAME'],
    :password   => ENV['IMAP_PASSWORD'],
    :port       => 993,
    :enable_ssl => true
  }

  delivery_method :smtp, {
    :address        => ENV['SMTP_ADDRESS'],
    :user_name      => ENV['SMTP_USERNAME'] || ENV['IMAP_USERNAME'],
    :password       => ENV['SMTP_PASSWORD'] || ENV['IMAP_PASSWORD'],
    :port           => 587,
    :authentication => 'plain',
    :enable_starttls_auto => true
  }
end

task :cron do
  urls = []
  emails = []

  mailboxes = []
  Mail.connection do |imap|
    mailboxes = imap.list('', '*').map(&:name).select { |n| n =~ /Unsubscribe$/ }
  end

  mailboxes.each do |mailbox|
    Mail.find(:mailbox => mailbox, :count => 100) do |mail, imap, message_id|
      if field = mail.header['List-Unsubscribe']
        if url = field.value[/<(https?:\/\/[^>]+)>/, 1]
          urls << url.to_s
        elsif uri = field.value[/<(mailto:[^>]+)>/, 1]
          uri = URI.parse(uri)
          subject_header = uri.headers.assoc('subject')
          subject = subject_header ? subject_header[1] : "Unsubscribe"
          from = mail.header['X-Delivered-to'] || mail.header['To']
          emails << [uri.to, from.to_s, subject]
        end
      end
      imap.uid_store(message_id, '+FLAGS', [:Seen, :Deleted])
    end
  end

  urls.uniq.each do |url|
    begin
      puts url
      open(url)
    rescue Exception => e
      warn e
    end
  end

  emails.uniq.each do |to, from, subject|
    begin
      Mail.deliver do
        to to
        from from
        subject subject
      end
    rescue Exception => e
      warn e
    end
  end
end

task :default => :cron
