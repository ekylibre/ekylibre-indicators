require 'rubygems'
require 'bundler/setup'
require 'net/imap'
require 'open-uri'
require 'base64'

require 'nokogiri'
require 'tty'
require 'tty-prompt'
require 'byebug'

task :pad do
  puts ""
end

task :fetch_exception_mails do
  pastel = Pastel.new
  imap = Net::IMAP.new('group.ekylibre.com', ssl: true, certs: nil, verify: false)
  prompt = TTY::Prompt.new
  logged_in = false
  skip = false

  while !logged_in && !skip
    puts "\n== BlueMind Exception Mails :"
    login = prompt.ask "Login : (___@ekylibre.com):\t"
    password = prompt.mask "Password:\t\t\t"

    begin
      imap.login("#{login}@ekylibre.com", password)
      logged_in = true
      next
    rescue
      puts "Invalid login or password, please retry."
      skip = prompt.yes?("Do you want to skip this step?", default: false)
    end
  end

  if skip
    puts pastel.underline("\nSkipping Exceptions mails indicator.")
  else
    folders = imap.list("", "*").map(&:name)
    folder = prompt.select "Folder to search?", folders

    imap.select(folder)
    messages = imap.search(["SENTSINCE", (Time.now - 7*24*60*60).strftime("%d-%b-%Y")])
    count = messages.count.to_f
    bar = TTY::ProgressBar.new("Fetching messages [:bar :percent]", width: 20, total: count)
    subjects = messages.each_with_index.map do |m,i|
      bar.advance
      imap.fetch(m, "ENVELOPE")[0].attr["ENVELOPE"]
    end
    subjects = subjects.select { |e| "#{e.from.first.mailbox}@#{e.from.first.host}" == "notifications@ekylibre.com" }
    if ENV['EXCLUDE_TENANT_NOT_FOUND']
      subjects = subjects.reject { |e| Base64.decode64(e.subject.split('?')[-2]) =~ /Could not find schema/ }
    end

    print "\r                                         "
    puts pastel.underline("\nNew Exception mails :\t\t#{pastel.bold(subjects.count.to_s)}")
    puts pastel.underline("\tout of which\t\t#{pastel.bold(subjects.map(&:subject).uniq.count.to_s)} unique ones")
  end
end

task :fetch_opened_issues_count do
  pastel = Pastel.new
  page = open("https://github.com/ekylibre/ekylibre/pulse")
  html = Nokogiri::HTML.parse(page.read)
  summary = html.css(".summary-stats")
  open_issues_number = summary.at("a:contains('New Issue')").css(".num").text.strip
  puts pastel.underline("Issues opened this week :\t#{pastel.bold(open_issues_number)}")
end

task :fetch_closed_issues_count do
  pastel = Pastel.new
  page = open("https://github.com/ekylibre/ekylibre/pulse")
  html = Nokogiri::HTML.parse(page.read)
  summary = html.css(".summary-stats")
  closed_issues_number = summary.at("a:contains('Closed Issue')").css(".num").text.strip
  puts pastel.underline("Issues closed this week :\t#{pastel.bold(closed_issues_number)}")
end

task :fetch_commits_count do
  pastel = Pastel.new
  page = open("https://github.com/ekylibre/ekylibre/pulse")
  html = Nokogiri::HTML.parse(page.read)
  summary = html.css(".diffstat-summary")
  commit_number = summary.at("strong:contains('commit')").css("span").text
  puts pastel.underline("Commits made this week :\t#{pastel.bold(commit_number)}")
end

task :fetch_commit_authors do
  pastel = Pastel.new
  page = open("https://github.com/ekylibre/ekylibre/pulse")
  html = Nokogiri::HTML.parse(page.read)
  summary = html.css(".diffstat-summary")
  author_number = summary.at("strong:contains('authors')").inner_html.split.first
  puts pastel.underline("Commit authors this week :\t#{pastel.bold(author_number)}")
end

task fetch_indicators: [:pad, :fetch_opened_issues_count, :fetch_closed_issues_count, :fetch_commits_count, :fetch_commit_authors, :fetch_exception_mails] do
  puts "\n\t\t\t\tFetched #{Time.now.strftime('%d-%b-%Y @ %k:%M')}"
end

task default: :fetch_indicators
