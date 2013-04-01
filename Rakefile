require 'mail'
require 'open-uri'

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
        elsif to = field.value[/<mailto:([^>]+)>/, 1]
          from = mail.header['X-Delivered-to'] || mail.header['To']
          emails << [to.to_s, from.to_s]
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

  emails.uniq.each do |to, from|
    begin
      Mail.deliver do
        to to
        from from
        subject 'Unsubscribe'
      end
    rescue Exception => e
      warn e
    end
  end
end

task :default => :cron
