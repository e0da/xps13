module Kernel

  def deep_freeze
    Module.nesting.first.send(:_deep_freeze, self)
  end

  private_class_method
  def self._deep_freeze(obj)
    obj.each {|o| _deep_freeze(o)} if obj.respond_to? :each
    obj.freeze if obj.respond_to? :freeze
  end
end

module Helpers

  # Map convenient keys to source/target pairs for symlinks
  # e.g. my_key: ['path/to/source', 'path/to/target']
  LINK_DEFINITIONS = {
    improve_palm_detection: [
      './lib/static_files/improve_palm_detection.xorg.conf',
      '/etc/X11/xorg.conf.d/50-synaptics.conf',
    ],
  }.deep_freeze

  COMMAND_BASE = '%{prefix}'.freeze
  LINK_COMMAND = "#{COMMAND_BASE}ln -sf %{source} %{target}".freeze
  MKDIR_COMMAND = "#{COMMAND_BASE}mkdir -p %{dir}".freeze

  class Link
    attr_accessor :source, :target
    def initialize(source, target)
      self.source, self.target = source, target
    end
  end

  def _link(key)
    source, target = links[key].source, links[key].target
    sudo_if_necessary { %{mkdir -p "#{File.dirname(target)}"} }
    sudo_if_necessary { %{ln -sf "#{source}" "#{target}"} }
  end

  private

  def ensure_directory_exists(dir)
  end

  def links
    @_links ||= LINK_DEFINITIONS.reduce({}) do |memo, (key, (source, target))|
      memo[key] = Link.new(File.expand_path(source), File.expand_path(target))
      memo
    end
  end

  def sudo_if_necessary
    begin
      sh yield
    rescue RuntimeError => error
      sh "sudo #{yield}" if /Command failed/ =~ error.message
    end
  end
end

self.class.class_eval { include Helpers }

desc 'Improve palm detection (requires X restart)'
task :improve_palm_detection do
  _link :improve_palm_detection
end
