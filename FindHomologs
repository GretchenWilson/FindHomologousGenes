## Command line program with input usage: "python FindHomologs.py refseq-file blast-file output-file"
## Program parses a file containing specific genes of interest within a given reference genome.
## After performing a BLAST analysis on the reference genome and the query genome, the resulting file is
##  parsed for the genes of interest which are reported in a tab delimited format.

import sys
import re 

class BlastHomologs(object):

    def __init__(self, reference_file, blast_file, output_file):
        self.reference_file = reference_file
        self.blast_file = blast_file
        self.output_file = output_file
        self.refseq = []

    def find_interest_genes(self):
        # parse through file containing the wanted genes and appends them
        # to a list called refseq

        rfh = open(self.reference_file)
        for line in rfh:
            line = line.strip()
            self.refseq.append(line)
            rfh.close()

    def find_homologs(self):
        bfh = open(blast_file)
        ofh = open(output, 'w')

        header = "Mouse_Refseq\tHuman_Refseq\tQuery_Length\tSearch_Length\tAlignment\tMatch\tBit_Score\tE_Score"
        print >>ofh, header

        #RegEx patterns for items of interest
        Signif_search = re.compile("^ref\|(.+)\|")
        M_id_search = re.compile("Query=.+ref\|(.+)\|")
        H_id_search = re.compile(">ref\|(.+)\|")
        M_len_search = re.compile("\((\d+)\s*letters\)")
        H_len_search = re.compile("Length\s*=\s*(\d+)")
        Score_search = re.compile("Score\s*=\s*(.+)\s*bits.+Expect\s*=\s*(.+)")
        Align_match_search = re.compile("Identities\s*=\s*(\d+)\/(\d+)")


        data = m_name = h_name = m_len = h_len = e_val = bit_score = align = match = None
        signif_align = []  # keeps a list of the significant alignments per gene

        for line in bfh:
            line = line.strip()
    
            if line.startswith("Query="): # start of a new mouse gene end of the previous gene
                id_match = M_id_search.search(line)

                if match:  # prints the previous gene
                    data = data = "%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s" % (m_name, h_name, m_len, h_len, align, match, bit_score, e_val)
                    print >>ofh, data
                    data = m_name = h_name = m_len = h_len = e_val = bit_score = align = match = None
                    signif_align = []

                if id_match:
                    option = id_match.group(1)
            
                    # check to see if mouse gene is a gene of interest
                    if option in self.refseq:
                        m_name = option

            elif m_name:

                if "letters" in line:
                    len_match = M_len_search.search(line)

                    if len_match:
                        m_len = len_match.group(1)

                elif line.startswith("ref"):
                    significant_match = Signif_search.search(line)

                    if significant_match:  # add all significant alignments to a list to reference later
                        option = significant_match.group(1)
                        signif_align.append(option)

                elif line.startswith(">"):  # looks for the start of a new alignment
                    id_match = H_id_search.search(line)

                    if match:  # print previous result
                        data = "%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s" % (m_name, h_name, m_len, h_len, align, match, bit_score, e_val)
                        print >>ofh, data
                        data = h_name = h_len = e_val = bit_score = align = match = None

                    if id_match:
                        option = id_match.group(1)
                        if option in signif_align:
                            h_name = option

                elif h_name:  # checks to see if significant alignment gene and procedes with >ref
                
                    if "Length" in line:
                        len_match = H_len_search.search(line)

                        if len_match:
                            h_len = len_match.group(1)

                    elif  "Score" in line:
                        score_match = Score_search.search(line)

                        if score_match:
                            score = float(score_match.group(1))

                            # if no bit score has been logged set current to the bit score
                            # for each alignment after, only procede if bit score is higher than current bit score
                            if not bit_score or score > bit_score:
                                bit_score = score
                                e_val = score_match.group(2)

                    elif e_val and "Identities" in line:
                        align_match = Align_match_search.search(line)
                
                        if align_match:
                            align = align_match.group(2)
                            match = align_match.group(1)

        bfh.close()
        ofh.close()

        if ofh:
            print "File parse complete. See results in %s" % self.output_file


if len(sys.argv) < 4:
    print "USAGE: python %s refseq-file blast-file output-file" % (sys.argv[0])
    sys.exit()

refseq_file = sys.argv[1]
blast_file = sys.argv[2]
output = sys.argv[3]
